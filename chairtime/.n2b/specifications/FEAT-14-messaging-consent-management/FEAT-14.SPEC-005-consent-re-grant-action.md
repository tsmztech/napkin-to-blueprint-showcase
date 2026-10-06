---
document_type: spec
spec_type: automation
spec_id: FEAT-14.SPEC-005
spec_name: Consent Re-Grant Action
spec_slug: consent-re-grant-action
parent_feature: FEAT-14
parent_feature_name: Messaging Consent Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Consent Re-Grant Action

## Overview

**Name:** Consent Re-Grant Action
**ID:** FEAT-14.SPEC-005
**Type:** Automation
**Purpose:** Records a client's opt-back-in to texting when they tap "Turn texting back on" on the Consent & Preferences screen.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

## Scope and Non-Goals

**In Scope:**
- Writing the Re-granted state to an existing Messaging Consent record when the client explicitly opts back in from FEAT-14.SPEC-001
- Resolving to the correct client-Pro relationship for the write

**Non-Goals:**
- Rendering the "Turn texting back on" button or its screen states -- owned by FEAT-14.SPEC-001; this automation is what that screen triggers.
- Re-granting as part of a later booking's choice -- owned by FEAT-14.SPEC-003, which handles the booking-time path to the same Re-granted-equivalent outcome (recorded as Granted there, since it is a fresh booking-time choice rather than an in-app toggle); this automation is exclusively the FEAT-14.SPEC-001 screen's action.
- Deciding precedence when this automation's write races a concurrent revoke -- owned by FEAT-14.SPEC-006; this automation performs its write unconditionally and defers to that spec's rule for the final persisted state.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client taps "Turn texting back on" | FEAT-14.SPEC-001 (Consent & Preferences) | Fires when the button is tapped, which is shown only when the client's current Messaging Consent state for this Pro is Revoked | The client-Pro relationship reference (from the authenticated viewing session), the current timestamp |

## Processing Logic

1. Receive the trigger from FEAT-14.SPEC-001, carrying the client-Pro relationship reference already scoped by that screen's viewing session.
2. Read the existing Messaging Consent record for this relationship.
3. Set the record's state to Re-granted and its timestamp to the current processing time. Consent_wording, channel, and phone_number are left unchanged -- a re-grant does not re-capture new consent wording, since the original booking-time or opt-back-in disclosure already covers ongoing messages once texting is active again.
4. Confirm the write completed so that FEAT-14.SPEC-001 can update its displayed status and any subsequent message for this relationship re-evaluates its channel through FEAT-08.SPEC-011.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Re-grant succeeds | The write completes normally | Messaging Consent state set to Re-granted, timestamp updated | FEAT-14.SPEC-001 shows "Texting turned back on" and updates the status line to "Texting: on" | FEAT-14.SPEC-001, FEAT-08.SPEC-011 |
| Re-grant superseded by a concurrent revoke | A STOP reply or opt-out link tap resolves with a later timestamp for the same relationship at effectively the same moment | Messaging Consent's final state is Revoked, per FEAT-14.SPEC-006's precedence rule, even though this automation's own write attempt succeeded in isolation | FEAT-14.SPEC-001 shows "Texting: off" on the client's next view, despite the "Texting turned back on" confirmation having appeared at the moment of the tap | FEAT-14.SPEC-006, FEAT-14.SPEC-001 |
| Write failure | The write itself cannot complete (a processing error) | No data change | FEAT-14.SPEC-001 shows "Couldn't update your preference. Try again." and the button remains available | FEAT-14.SPEC-001 |

## Data Model

**Reads:** Messaging Consent -- the existing record for the resolved relationship.
**Creates:** None -- this automation only updates an existing record, which always already exists by the time a client can reach FEAT-14.SPEC-001 (created no later than their first booking, per FEAT-14.SPEC-003).
**Updates:** Messaging Consent -- state set to Re-granted, timestamp updated.
**Deletes:** None.

## Business Rules

- A re-grant takes effect immediately for the client's next message, mirroring XBR-15's same immediacy for a revoke -- there is no grace period or delay in either direction.
- Re-granted is treated identically to Granted by every downstream consumer of consent state (FEAT-08.SPEC-011, FEAT-14.SPEC-007) -- it is not a lesser or probationary tier of consent.
- This automation never re-captures consent_wording -- the wording field remains the historical record of the client's original opt-in disclosure, consistent with FEAT-14.SPEC-004's identical treatment of that field on a revoke.
- When this automation's write and a concurrent revoke could both apply to the same record, FEAT-14.SPEC-006 determines the final persisted state; this automation always performs its own write and does not itself compare timestamps against a competing write.

## Edge Cases

- **The client's Messaging Consent record is already Granted or Re-granted when this automation fires (a stale screen state -- the button should not have been shown)** -- The write proceeds harmlessly: state is set to Re-granted (unchanged in practical effect from Granted) and the timestamp updates; no error is raised, since re-affirming active consent is not a meaningful failure.
- **A STOP reply for the same relationship arrives within the same second as this automation's trigger** -- Concurrent trigger firing: both automations write independently; FEAT-14.SPEC-006 resolves which timestamp is later and, if genuinely indistinguishable, defaults to the no-text (Revoked) state.
- **The client double-taps "Turn texting back on" before the first write completes** -- FEAT-14.SPEC-001's screen-level debounce (button disabled while loading) prevents a second trigger from firing while the first is in flight; if a second trigger were to reach this automation regardless, it would find the record already Re-granted and proceed as a harmless re-affirming write, identical to the stale-state case above.
- **The client's phone number changed and their consent was invalidated (FEAT-14.SPEC-008) before they attempt this re-grant** -- FEAT-14.SPEC-001 reflects the invalidated (Revoked) state and still offers the button, since from the client's perspective texting is off; this automation's write still succeeds, setting state to Re-granted, but the phone_number field is left unchanged from the invalidated record -- the client's next booking (FEAT-14.SPEC-003) is what updates phone_number to the current number, since this screen-triggered automation has no phone-number input of its own to write a new value from.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-001 (Consent & Preferences) | Triggered by (inbound) | The "Turn texting back on" button fires this automation |
| FEAT-14.SPEC-001 (Consent & Preferences) | Affects (outbound) | The screen's status line and confirmation reflect this automation's outcome |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | Affects (outbound) | Reads the resulting Re-granted state for the client's next message |
| FEAT-14.SPEC-006 (Concurrent Consent Update Resolution) | References (inbound) | Governs the final state when this automation's write races a concurrent revoke |
| FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule) | References (inbound) | Explains why the button can be shown even after a number change invalidated consent |

## Analytics and Success Signals

- **consent_regranted** (was_superseded_by_revoke: yes / no) -- N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature or references re-grant volume; retained as an operational signal so opt-back-in activity is observable.

## Acceptance Criteria

**FEAT-14.SPEC-005-AC-01:** Given Riley's Messaging Consent with Talia is Revoked, when she taps "Turn texting back on" on FEAT-14.SPEC-001, then her record's state is set to Re-granted with the current timestamp.

**FEAT-14.SPEC-005-AC-02:** Given Riley's re-grant write succeeds, when FEAT-14.SPEC-001 receives the outcome, then it shows "Texting turned back on" and updates its status line to "Texting: on".

**FEAT-14.SPEC-005-AC-03:** Given Riley's re-grant write completes, when the very next message for her booking with Talia is about to send, then FEAT-08.SPEC-011 selects text as the channel.

**FEAT-14.SPEC-005-AC-04:** Given a STOP reply for Riley's relationship with Talia arrives within the same second as her re-grant tap with a later timestamp, when both writes complete, then FEAT-14.SPEC-006 resolves the final state to Revoked.

**FEAT-14.SPEC-005-AC-05:** Given Riley's record is already Granted when this automation fires due to a stale screen state, when the write proceeds, then no error is raised and the state is set to Re-granted without disrupting the client experience.

**FEAT-14.SPEC-005-AC-06:** Given the re-grant write fails due to a processing error, when the failure is reported, then FEAT-14.SPEC-001 shows "Couldn't update your preference. Try again." and the button remains available.

**FEAT-14.SPEC-005-AC-07:** Given Riley's phone number was changed and her consent invalidated per FEAT-14.SPEC-008, when she taps "Turn texting back on", then the write succeeds setting state to Re-granted, but phone_number is left unchanged pending her next booking.

**FEAT-14.SPEC-005-AC-08:** Given this automation never re-captures consent wording, when a record's state changes to Re-granted, then its consent_wording field remains exactly as originally captured by FEAT-14.SPEC-003.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (succeeds, superseded, write failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
