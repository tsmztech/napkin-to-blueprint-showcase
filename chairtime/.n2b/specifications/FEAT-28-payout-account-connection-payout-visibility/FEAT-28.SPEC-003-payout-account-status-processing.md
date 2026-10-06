---
document_type: spec
spec_type: automation
spec_id: FEAT-28.SPEC-003
spec_name: Payout Account Status Processing
spec_slug: payout-account-status-processing
parent_feature: FEAT-28
parent_feature_name: Payout Account Connection & Payout Visibility
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Payout Account Status Processing

## Overview

**Name:** Payout Account Status Processing
**ID:** FEAT-28.SPEC-003
**Type:** Automation
**Purpose:** Creates and updates the Payout Account record from the payment-processing capability's reported status changes, drives the go-live gate signal (XBR-06), and triggers the status notification.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility

## Scope and Non-Goals

**In Scope:**
- Creating the Payout Account record the moment the capability first reports a connection outcome
- Updating the Payout Account's status field on every subsequent status report (Verification Pending -> Active -> Action Required -> Disconnected)
- Signaling the go-live gate (XBR-06) whenever status changes to or from Active
- Triggering FEAT-28.SPEC-007 (Payout Status Notification) on the two status transitions this feature owns

**Non-Goals:**
- Requesting or receiving the status report itself from the capability -- owned by FEAT-28.SPEC-006 (Payout Account Connection & Verification); this automation only processes what that integration hands it
- Deciding whether a candidate Payout Account is eligible (one per Pro, country/currency match) -- owned by FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints); this automation applies that spec's rule at creation time rather than re-deriving it
- Displaying the resulting status -- owned by FEAT-28.SPEC-001 (first connection) and FEAT-28.SPEC-002 (ongoing dashboard); this automation only writes the record those screens read
- Refund processing or money-list composition -- owned by FEAT-09/FEAT-30's own integration specs and FEAT-28.SPEC-005 respectively; this automation governs only the Payout Account entity's own status

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Connection outcome first reported | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Fires the first time the capability reports any outcome (Verification Pending or Active) for a Pro Account with no existing Payout Account | Pro Account reference, processor_account_reference, reported status, country, currency |
| Status change reported | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Fires whenever the capability reports a status different from the Payout Account's current status | Payout Account reference, new status, event time, and (for Action Required) the capability's reason |
| Resolution outcome reported | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Fires when Talia completes the processor's own resolution flow for an Action Required account and the capability reports the outcome | Payout Account reference, new status (Active or still Action Required), event time |

## Processing Logic

1. Receive the reported outcome from FEAT-28.SPEC-006, carrying the Pro Account reference, the reported status, and the event time the capability assigns to the report.
2. If no Payout Account exists yet for this Pro Account, create one: set processor_account_reference, status to the reported value, and country/currency from the report, after confirming eligibility per FEAT-28.SPEC-004 (one account per Pro, country/currency match to the Pro Account).
3. If a Payout Account already exists, compare the reported event time to the Payout Account's last-recorded event time. If the reported event is not newer, discard it (see Edge Cases -- out-of-order delivery); otherwise proceed.
4. Update the Payout Account's status field to the reported value.
5. If the new status is Active and the previous status was not Active, signal the go-live gate (XBR-06) that this Pro's payout precondition is now satisfied, and trigger FEAT-28.SPEC-007 with the "verification complete" content.
6. If the new status is Action Required, signal the go-live gate that the precondition is not satisfied (if it was previously satisfied, existing bookings and the booking link's live state are unaffected per the Brief's Alternate flow -- only new go-live evaluations are blocked), and trigger FEAT-28.SPEC-007 with the "needs action" content, carrying the capability's reason.
7. If the new status is Disconnected, signal the go-live gate that the precondition is not satisfied.
8. Record the status change as an Activity Event via FEAT-16 (Booking & Payment Activity Record), per XBR-22's evidence trail for status-change signals.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Payout Account created (pending) | First-ever report is Verification Pending | Payout Account created with status Verification Pending | FEAT-28.SPEC-001 shows the pending status | FEAT-28.SPEC-001, FEAT-28.SPEC-002 |
| Payout Account created (active) | First-ever report is Active | Payout Account created with status Active | FEAT-28.SPEC-001 shows Active; go-live gate satisfied; FEAT-28.SPEC-007 fires | FEAT-28.SPEC-001, FEAT-28.SPEC-002, FEAT-28.SPEC-007, FEAT-15 (go-live evaluation) |
| Status updated to Active | Existing Payout Account moves from Verification Pending or Action Required to Active | Payout Account.status set to Active | FEAT-28.SPEC-002's banner clears; go-live gate satisfied; FEAT-28.SPEC-007 fires with "verification complete" | FEAT-28.SPEC-002, FEAT-28.SPEC-007, FEAT-15 |
| Status updated to Action Required | Existing Payout Account (any prior status) receives an Action Required report | Payout Account.status set to Action Required; reason recorded | FEAT-28.SPEC-002 shows the prominent banner; go-live gate signals not-satisfied for future evaluations; FEAT-28.SPEC-007 fires with "needs action" | FEAT-28.SPEC-002, FEAT-28.SPEC-007, FEAT-15 |
| Status updated to Disconnected | The capability reports the account disconnected | Payout Account.status set to Disconnected | FEAT-28.SPEC-002 shows an equivalent needs-action treatment; go-live gate signals not-satisfied | FEAT-28.SPEC-002, FEAT-15 |
| Stale/out-of-order report discarded | Reported event time is not newer than the Payout Account's last-recorded event time | None | No user feedback -- the discard is silent, per the processor-authoritative resolution rule | None |
| Report for a non-existent, ineligible Payout Account | FEAT-28.SPEC-004's eligibility check fails at creation time (e.g., a second account attempted for the same Pro) | No Payout Account created or updated | FEAT-28.SPEC-001/FEAT-28.SPEC-006 show the eligibility rejection message from FEAT-28.SPEC-004 | FEAT-28.SPEC-004 |

## Data Model

**Reads:** Pro Account -- country, currency (for the eligibility check at creation, per FEAT-28.SPEC-004).
**Creates:** Payout Account -- processor_account_reference, status, country, currency, on first-ever reported outcome for a Pro Account.
**Updates:** Payout Account -- status (and, for Action Required, the recorded reason), on every subsequent reported status change.
**Deletes:** None -- this automation never deletes a Payout Account; disconnection is represented as a status value, not a deletion, consistent with the dependency map's Payout Account lifecycle.

## Business Rules

- The processor-reported status is authoritative and resolved last-write-wins by the processor's own event time (dependency map, Payout Account Contention) -- this automation never lets a locally-cached or stale report overwrite a newer one.
- XBR-06: Active is the only status from which a deposit can be taken; this automation is the sole writer of that status, so FEAT-07's own eligibility check (FEAT-28.SPEC-004) always reads a value this automation last set.
- FEAT-28.SPEC-004 governs eligibility at Payout Account creation (one per Pro, country/currency match); this automation defers to that spec's rule rather than re-implementing it.
- XBR-22: every Payout Account status change is written to the append-only activity record (FEAT-16), giving the Pro and Support a durable trail of when the account's standing changed.
- XBR-26: a status change to or from Active is exactly the signal FEAT-15's go-live check consumes; this automation never itself decides whether the booking link goes live -- it only reports the precondition's current truth.

## Edge Cases

- **The capability reports the same status twice in a row (e.g., two Active reports)** -- The second report is a no-op: status is already Active, no further go-live signal or notification fires, per Deduplication in FEAT-28.SPEC-007.
- **Two status reports for the same Payout Account arrive out of order (an older Active report arrives after a newer Action Required report)** -- The event-time comparison in Processing Logic step 3 discards the stale Active report; the Payout Account remains Action Required, matching the capability's true, more recent state.
- **A report arrives for a Pro Account that has since closed (FEAT-29)** -- The report is still applied to the Payout Account record (which is soft-removed only as part of the 30-day cooling-off closure, per the Brief's CRUD matrix), but no notification fires since there is no active Pro session to notify.
- **Concurrent trigger firing (two status reports for the same Payout Account arrive at effectively the same time)** -- Each is processed against its own event time independently; the one with the later event time wins regardless of arrival order, per the last-write-wins resolution rule, so the two reports never race to an inconsistent final state.
- **A status report arrives while a previous report for the same Payout Account is still being processed** -- Processing for one Payout Account is serialized: the second report waits for the first to finish applying its status change before its own event-time comparison runs, so no two updates to the same record are ever applied out of the intended order.
- **The very first report for a brand-new Payout Account is itself Action Required (e.g., a bank detail rejected on first submission)** -- The Payout Account is still created (per Outcome Definitions), with status Action Required from creation; FEAT-28.SPEC-001 shows this exactly as it shows a later-arising Action Required condition.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Triggered by (inbound) | Every reported outcome and status change originates from that integration's Inbound Events |
| FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints) | References (inbound) | Eligibility rule applied at Payout Account creation |
| FEAT-28.SPEC-001 (Payout Account Connection) | Affects (outbound) | Displays the first recorded status |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | Affects (outbound) | Displays the ongoing status and its banner |
| FEAT-28.SPEC-007 (Payout Status Notification) | Triggers (outbound) | Fired on transitions to Active and to Action Required |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) -- within FEAT-15 (Pro Onboarding & Setup Wizard) | Affects (outbound) | Consumes the go-live gate signal (XBR-06, XBR-26) |
| FEAT-07 (Deposit Payment at Booking) | Affects (outbound) | Reads the Active status this automation maintains before any deposit charge (XBR-06) |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | Every status change is recorded as an Activity Event (XBR-22) |

## Analytics and Success Signals

- **payout_account_created** (initial status: verification_pending / active) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **payout_account_activated** (previous status) -- supports success-metrics.md: "Payout Transparency"
- **payout_account_action_required** (reason category) -- supports success-metrics.md: "Payout Transparency"
- **payout_account_status_report_discarded** (reason: stale_event_time / ineligible) -- N/A -- no Stage 2 metric measures discarded or rejected status reports; retained so the correctness bar (ASMP-26) for this automation's processor-authoritative rule is observable.

## Acceptance Criteria

**FEAT-28.SPEC-003-AC-01:** Given Talia has no Payout Account yet, when the capability first reports Verification Pending, then a Payout Account is created with that status.

**FEAT-28.SPEC-003-AC-02:** Given Talia has no Payout Account yet, when the capability first reports Active, then a Payout Account is created with status Active, the go-live gate is signaled satisfied, and FEAT-28.SPEC-007 fires.

**FEAT-28.SPEC-003-AC-03:** Given Talia's Payout Account is Verification Pending, when the capability reports Active, then status updates to Active, the go-live gate is signaled satisfied, and FEAT-28.SPEC-007 fires with the "verification complete" content.

**FEAT-28.SPEC-003-AC-04:** Given Talia's Payout Account is Active, when the capability reports Action Required, then status updates to Action Required, the go-live gate is signaled not-satisfied for future evaluations, and FEAT-28.SPEC-007 fires with the "needs action" content and reason.

**FEAT-28.SPEC-003-AC-05:** Given Talia's Payout Account is Action Required, when she resolves it through the processor's flow and the capability reports Active, then status updates to Active and FEAT-28.SPEC-007 fires again with the "verification complete" content.

**FEAT-28.SPEC-003-AC-06:** Given the capability reports Disconnected, when the report is processed, then status updates to Disconnected and the go-live gate is signaled not-satisfied.

**FEAT-28.SPEC-003-AC-07:** Given Talia's Payout Account is already Active, when a second Active report arrives, then nothing changes and no duplicate notification fires.

**FEAT-28.SPEC-003-AC-08:** Given two reports for the same Payout Account arrive out of order, when the older report's event time is earlier than the account's current recorded event time, then the older report is discarded and the account reflects the newer report's status.

**FEAT-28.SPEC-003-AC-09:** Given a candidate second Payout Account is reported for a Pro who already has one, when FEAT-28.SPEC-004's eligibility check runs, then no new Payout Account is created and the rejection is surfaced through FEAT-28.SPEC-001 or FEAT-28.SPEC-006.

**FEAT-28.SPEC-003-AC-10:** Given a status change is processed, when the write completes, then an Activity Event is recorded for that Payout Account per XBR-22.

**FEAT-28.SPEC-003-AC-11:** Given a status report arrives for a Pro Account that has since closed, when the report is processed, then the Payout Account record is updated but no notification is delivered.

**FEAT-28.SPEC-003-AC-12:** Given two status reports for the same Payout Account arrive at effectively the same time, when both are processed, then the one with the later event time is the account's final state regardless of arrival order.

**FEAT-28.SPEC-003-AC-13:** Given a report arrives for a Payout Account while a previous report for that same account is still being applied, when the second report's turn comes, then its event-time comparison runs only after the first report's update has fully applied.

**FEAT-28.SPEC-003-AC-14:** Given a brand-new Payout Account's very first report is Action Required, when it is processed, then the account is created directly with status Action Required and FEAT-28.SPEC-007 fires with the "needs action" content.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (first outcome, status change, resolution outcome) | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
