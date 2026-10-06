---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-007
spec_name: Proposal Resend
spec_slug: proposal-resend
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 7
---

# Automation Spec: Proposal Resend

## Overview

**Name:** Proposal Resend
**ID:** FEAT-02.SPEC-007
**Type:** Automation
**Purpose:** Re-sends the link for an already-Sent proposal without creating a new version, for the case where Owen simply cannot find the original email.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Re-triggering delivery of the current Sent version's link to the client's Primary contact(s)
- Rate-limiting rapid repeated resends

**Non-Goals:**
- Creating a new proposal version or changing its content -- that is FEAT-02.SPEC-006 (Void & Resend), triggered only by an actual content edit; this automation never alters scope, price, or currency.
- Composing the email content -- owned by FEAT-02.SPEC-011 (Proposal Sent/Resent Email); this automation only triggers delivery of the existing version's content.
- Resending a Draft, Voided, or Accepted proposal -- resend applies only to a currently Sent proposal, since those are the only statuses awaiting the client's decision.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia taps "Resend" on the Detail screen | FEAT-02.SPEC-003 (Proposal Detail) | The proposal is currently in Sent status | Proposal id, existing scope_description, price, currency, sent_at, owning client's Primary contact(s) |

## Processing Logic

1. Read the Proposal's current status; if it is not Sent, stop and report a state-mismatch failure (see Edge Cases).
2. Check the resend rate limit for this proposal (see Business Rules); if exceeded, stop and report the rate-limit outcome.
3. Confirm the client still has at least one Primary contact (XBR-07); if not, stop and report the Primary-contact failure.
4. Trigger FEAT-02.SPEC-011 (Proposal Sent/Resent Email) to the client's current Primary contact(s), carrying the existing version's content unchanged.
5. Record the resend event (timestamp) against the proposal for rate-limit tracking.
6. Write an Activity Log Entry recording the resend (XBR-05), owned by FEAT-13.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Resend succeeds | Proposal is Sent, rate limit not exceeded, client has a Primary contact | Resend event recorded for rate-limit tracking; no change to proposal content, status, or sent_at | Toast on FEAT-02.SPEC-003: "Proposal link resent to {Primary Contact name}." | FEAT-02.SPEC-003, FEAT-02.SPEC-011, FEAT-13 |
| Blocked -- state mismatch | The proposal is no longer Sent (e.g., already voided, discarded, or accepted since the Detail screen loaded) | None | Detail screen re-fetches and shows the current status; the Resend action is not available in that status | FEAT-02.SPEC-003 |
| Blocked -- rate limited | A resend for this proposal was already sent within the rate-limit window | None | Toast: "You already resent this proposal recently. Try again in a few minutes." | FEAT-02.SPEC-003 |
| Blocked -- no Primary contact | Client's last Primary contact was removed since the original send | None | Detail screen shows: "This client has no Primary contact yet." with a link into FEAT-18 | FEAT-02.SPEC-003 |
| Failure | Processing error (e.g., connectivity lost mid-operation) | None | Toast: "Could not resend the proposal link. Try again." | FEAT-02.SPEC-003 |

## Data Model

**Reads:** Proposal -- status, scope_description, price, currency (unchanged, read-only here). Client -- Primary contact roster.
**Creates:** A resend-event record used for rate-limit tracking (not a new Proposal version). Activity Log Entry (via FEAT-13).
**Updates:** None on the Proposal record itself -- status, content, and sent_at are all untouched by a resend.
**Deletes:** None.

## Business Rules

- Resend never creates a new proposal version and never changes sent_at -- it only re-delivers the existing Sent version's link (distinguishing it from FEAT-02.SPEC-006, which resends because content changed).
- Resend is limited to one successful resend per proposal per platform parameter: `proposal-resend-cooldown-minutes`, to prevent accidental repeated sends from generating multiple emails for the same unchanged content.
- Resend requires the same client Primary-contact eligibility as the original send (XBR-07).

## Edge Cases

- **The proposal is voided or accepted between the Detail screen loading and Nadia tapping Resend** -- Reported as the state-mismatch outcome; Resend is not available once the Detail screen's next load reflects the new status.
- **Nadia taps Resend twice within the cooldown window** -- The second attempt is blocked with the rate-limited outcome; no second email is sent.
- **Concurrent trigger firing (Resend tapped from two open Detail sessions for the same proposal at effectively the same time)** -- Only the first passes the rate-limit check within the window; the second receives the rate-limited outcome. At most one email is delivered for the pair.
- **A trigger fires while a previous resend for the same proposal is still in flight** -- The rate-limit check (Business Rules) also functions as the in-flight guard: a resend in progress is treated as having just occurred, so a same-instant second trigger is rejected as rate-limited rather than double-sending.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-003 (Proposal Detail) | Triggered by (inbound) | "Resend" action on a Sent proposal triggers this automation |
| FEAT-02.SPEC-011 (Proposal Sent/Resent Email) | Affects (outbound) | Triggered on success to re-deliver the existing version's link |
| FEAT-18 (Client Contact Management & Roles) | References (outbound) | Primary-contact requirement checked against this feature's contact roster |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the resend event to the trail (XBR-05) |

## Analytics and Success Signals

- **proposal_resent** (project id, days since original sent_at) -- supports success-metrics.md: "Notification Delivery Reliability" (measures how often a client-facing email had to be manually re-delivered)
- **proposal_resend_rate_limited** (-- ) -- N/A -- no Stage 2 metric measures rate-limit hits; retained for operational visibility into resend friction

## Acceptance Criteria

**FEAT-02.SPEC-007-AC-01:** Given a proposal is Sent and the client has a Primary contact, when Nadia taps "Resend", then FEAT-02.SPEC-011 is triggered and the toast "Proposal link resent to {Primary Contact name}." appears, with the proposal's status, content, and sent_at unchanged.

**FEAT-02.SPEC-007-AC-02:** Given the proposal was voided since the Detail screen loaded, when Nadia taps "Resend", then the automation reports the state-mismatch outcome and the screen re-fetches to show the current status.

**FEAT-02.SPEC-007-AC-03:** Given Nadia resent the proposal moments ago, when she taps "Resend" again within the cooldown window, then the toast "You already resent this proposal recently. Try again in a few minutes." appears and no email is sent.

**FEAT-02.SPEC-007-AC-04:** Given the client's last Primary contact was removed since the original send, when Nadia taps "Resend", then "This client has no Primary contact yet." appears with a link into FEAT-18.

**FEAT-02.SPEC-007-AC-05:** Given a processing failure occurs during resend, then the toast "Could not resend the proposal link. Try again." appears.

**FEAT-02.SPEC-007-AC-06:** Given two Detail sessions trigger Resend for the same proposal at effectively the same time, then at most one email is delivered and the second trigger receives the rate-limited outcome.

**FEAT-02.SPEC-007-AC-07:** Given Resend succeeds, then an Activity Log Entry recording the resend is written (FEAT-13).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (success, state mismatch, rate limited, no primary contact, failure) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
