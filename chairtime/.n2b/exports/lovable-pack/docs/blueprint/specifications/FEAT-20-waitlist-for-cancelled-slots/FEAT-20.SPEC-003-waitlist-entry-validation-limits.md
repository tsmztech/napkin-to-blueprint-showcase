---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-20.SPEC-003
spec_name: Waitlist Entry Validation & Limits
spec_slug: waitlist-entry-validation-limits
parent_feature: FEAT-20
parent_feature_name: Waitlist for Cancelled Slots
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Waitlist Entry Validation & Limits

## Overview

**Name:** Waitlist Entry Validation & Limits
**ID:** FEAT-20.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs what a valid waitlist join looks like -- the service/date-range shape, the 3-active-entries-per-Pro cap, and the notice/horizon bounds a joined date range must respect.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots
**Governed Entity:** Waitlist Entry

## Scope and Non-Goals

**In Scope:**
- Field validation for every Waitlist Entry field set at creation
- The 3-active-entries-per-Pro cap (across Requested and Notified states)
- Minimum booking notice and booking horizon bounds applied to a joined date or date range (XBR-03)
- Authorization for who may create, read, and delete a Waitlist Entry

**Non-Goals:**
- The claim window, matching priority, and contention resolution once an entry is Notified -- owned by FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule); this spec governs only what makes a join valid at creation time, not what happens after
- Identity-field format rules (name, phone, email) -- those are ordinary contact-detail formats matching FEAT-05.SPEC-003's own conventions, enforced inline on FEAT-20.SPEC-001 as noted there, not a waitlist-specific rule this spec owns
- Deciding which entries match a freed slot -- owned by FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching); this spec only governs whether an entry was valid to create, not how it is later matched

## Governed Entity

**Entity:** Waitlist Entry
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| service | reference | The Service this entry is waitlisted for |
| date_mode | enum (single day, range) | Whether the entry names one day or a range of up to platform parameter: `waitlist-join-range-max-days` |
| start_date | date | The requested day, or the first day of a requested range |
| end_date | date | Equal to start_date for a single day; the last day of a requested range otherwise |
| state | enum (Requested, Notified, Converted, Expired) | The entry's current lifecycle state |
| claim_deadline | date/time | 30 minutes after notification; set only once the entry is Notified (owned by FEAT-20.SPEC-004/SPEC-005, not by this spec) |

**Referenced (read-only):** Service -- to confirm the requested service exists and is Active; Availability Rule (via FEAT-03) -- to confirm the notice and horizon bounds current at join time.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-20.SPEC-001 | Join Waitlist | On Join button submit -- the sole point this spec's rules are evaluated, since a Waitlist Entry is never edited after creation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| service | Must reference an existing, Active Service belonging to this Pro | Always | On submit | "This service is no longer available. Choose another service." | Yes |
| date_mode | Must be one of "single day" or "range" | Always | On submit | No user-facing error -- the screen's toggle only offers these two values | No |
| start_date | Required; must not be before the earliest day permitted by the Pro's current minimum booking notice (XBR-03, via FEAT-03) | Always | On submit | "The earliest day you can join for is {earliest_permitted_date}." | Yes |
| end_date | Required; equal to start_date in single-day mode; in range mode, must be on or after start_date and at most platform parameter: `waitlist-join-range-max-days` minus one day after it | Always | On submit | "A waitlist range can span at most {waitlist-join-range-max-days} days." | Yes |
| end_date | Must not be after the latest day permitted by the Pro's current booking horizon (XBR-03, via FEAT-03) | Always | On submit | "The latest day you can join for is {latest_permitted_date}, based on how far ahead this Pro takes bookings." | Yes |
| state | No validation beyond data type -- always set to Requested on creation by this spec; every later value is set exclusively by FEAT-20.SPEC-005/SPEC-006/SPEC-007 | Always | -- | -- | -- |
| claim_deadline | No validation beyond data type at creation -- always empty at creation; set only by FEAT-20.SPEC-005 when the entry transitions to Notified | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Date-range shape | date_mode, start_date, end_date | If date_mode is "single day," end_date must equal start_date; if "range," end_date must be on or after start_date and the span must not exceed platform parameter: `waitlist-join-range-max-days` | "A waitlist range can span at most {waitlist-join-range-max-days} days." |
| Active-entries cap | service (via the owning Client-Pro relationship), state | The requesting Client may not hold more than platform parameter: `waitlist-max-active-entries-per-pro` entries in state Requested or Notified with this Pro at the moment of a new join | "You're already on {waitlist-max-active-entries-per-pro} waitlists with this Pro. Leave one before joining another." |
| Notice/horizon bounds | start_date, end_date | The entire requested span (start_date through end_date) must fall within the window the Pro's current minimum booking notice and booking horizon permit (XBR-03), evaluated at the moment of join, not re-evaluated later as those settings may change | "This date range falls outside the times this Pro currently takes bookings." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a Waitlist Entry | The Client (Riley) | Always, subject to the field and cross-field rules above | The specific rule's error message from the tables above |
| Create a Waitlist Entry | The Pro (Talia) | Never -- the product defines no Pro-initiated waitlist join | No control exists for the Pro to create an entry on a client's behalf; this action is not offered anywhere in the Pro's account |
| Create a Waitlist Entry | Platform Operator (Support) | Never | No control exists for Support to create an entry |
| Read own Waitlist Entry (state, position/status) | The Client (Riley) | Only entries she herself created (Own-only, per the Access Matrix) | She never sees another client's entry |
| Read aggregate waitlist demand for a day | The Pro (Talia) | Always, as a count only, via FEAT-12 -- never individual entries or client identities | -- |
| Read individual Waitlist Entry state | Platform Operator (Support) | View-only, for troubleshooting a specific Pro's reported issue, via FEAT-19 only | -- |
| Delete (leave) a Waitlist Entry | The Client (Riley) | Only entries she herself created, in state Requested or Notified | -- (a Converted or Expired entry offers no Leave action per FEAT-20.SPEC-002, since it is already terminal) |
| Delete (leave) a Waitlist Entry | The Pro (Talia) | Never -- the Pro cannot remove a client's waitlist entry on her own initiative | No control exists for the Pro to remove an entry |
| Update any field after creation | Any role | Never -- per the Entity-Lifecycle Coverage Matrix, no field is ever edited after creation; only the state transitions FEAT-20.SPEC-005/SPEC-006/SPEC-007 own occur | No edit control exists anywhere in the product for a created entry's service or date range |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| state | Set to Requested | On creation, once all validation passes | No |
| service | Carried unmodified from the fully booked service page's context (FEAT-05.SPEC-002 -> FEAT-20.SPEC-001) | On creation | No -- the client cannot pick a different service on the Join Waitlist screen itself |
| claim_deadline | Left unset | On creation | No -- set only by FEAT-20.SPEC-005 when the entry is later Notified |

## Business Rules

- XBR-03 governs the notice/horizon bounds entirely: the Pro's current minimum booking notice and booking horizon (owned by FEAT-02) bound every client-facing booking path, including this one; a Pro-side change to those settings after an entry was created does not retroactively invalidate it (the entry's date range was valid when joined; only new joins are checked against current settings).
- The 3-active-entries-per-Pro cap (platform parameter: `waitlist-max-active-entries-per-pro`) counts only Requested and Notified entries -- Converted and Expired entries never count against it, and a client who leaves an entry immediately frees a slot in the cap for a new join.
- A join request is evaluated once, atomically, at submission -- there is no partial or draft Waitlist Entry state; either every rule passes and the entry is created in Requested, or none of it is created.
- This spec's rules apply identically regardless of date_mode -- a single-day join and a range join are evaluated by the same cross-field logic, with a single day simply being the degenerate one-day case of a range.

## Edge Cases

- **A requested range's start_date is exactly at the minimum-notice boundary** -- Passes; the boundary day itself is permitted, consistent with FEAT-02's own boundary-inclusive convention for notice and horizon.
- **A requested range's end_date is exactly platform parameter: `waitlist-join-range-max-days` minus one day after start_date** -- Passes, since this is the maximum permitted span, not one day beyond it.
- **A requested range's end_date is exactly platform parameter: `waitlist-join-range-max-days` after start_date** -- Rejected with the range-shape error, since this spans one day more than the maximum permitted.
- **The Client holds exactly platform parameter: `waitlist-max-active-entries-per-pro` minus one active entries and submits one more** -- Passes; this is the boundary case that reaches, but does not exceed, the cap.
- **The Client holds exactly the cap and submits one more** -- Rejected with the cap message.
- **The Pro's booking horizon shortens between when Riley started filling the form and when she submits** -- The submitted range is checked against the Pro's current horizon at the moment of submit, not at the moment the form was opened; a range that was valid when she started but is no longer valid at submit is rejected with the notice/horizon message, and she can adjust her requested dates and resubmit.
- **The requested Service is archived between when Riley arrived on the Join Waitlist screen and when she submits** -- Rejected with "This service is no longer available. Choose another service." per the service-existence rule; she is routed back to FEAT-05.SPEC-001's current service list.
- **A single-day join where start_date and end_date are somehow submitted as different values (a malformed submission)** -- Rejected by the date-range-shape rule, since single-day mode requires them to be equal; this is treated identically to any other range-shape violation.

## Acceptance Criteria

**FEAT-20.SPEC-003-AC-01:** Given Riley submits a join for an Active service with a valid single-day date within notice and horizon, when validation runs, then the entry is created in state Requested.

**FEAT-20.SPEC-003-AC-02:** Given Riley submits a join for a service that is not Active, when validation runs, then she sees "This service is no longer available. Choose another service." and no entry is created.

**FEAT-20.SPEC-003-AC-03:** Given Riley submits a range spanning exactly platform parameter: `waitlist-join-range-max-days` minus one day, when validation runs, then it passes as the maximum permitted span.

**FEAT-20.SPEC-003-AC-04:** Given Riley submits a range spanning platform parameter: `waitlist-join-range-max-days`, when validation runs, then she sees "A waitlist range can span at most {waitlist-join-range-max-days} days." and no entry is created.

**FEAT-20.SPEC-003-AC-05:** Given Riley's requested start_date falls before the Pro's current minimum booking notice, when validation runs, then she sees the earliest-permitted-date message and no entry is created.

**FEAT-20.SPEC-003-AC-06:** Given Riley's requested end_date falls beyond the Pro's current booking horizon, when validation runs, then she sees the latest-permitted-date message and no entry is created.

**FEAT-20.SPEC-003-AC-07:** Given Riley already holds platform parameter: `waitlist-max-active-entries-per-pro` minus one active entries with this Pro, when she submits one more valid join, then it succeeds, reaching the cap.

**FEAT-20.SPEC-003-AC-08:** Given Riley already holds platform parameter: `waitlist-max-active-entries-per-pro` active entries with this Pro, when she submits another, then she sees the cap message and no entry is created.

**FEAT-20.SPEC-003-AC-09:** Given Riley leaves one of her active entries and then submits a new join while still holding the cap minus one, then the new join succeeds, since the leave freed a slot in the cap.

**FEAT-20.SPEC-003-AC-10:** Given Talia (the Pro) looks for a way to create a waitlist entry on a client's behalf, then no such control exists anywhere in her account.

**FEAT-20.SPEC-003-AC-11:** Given Talia views her Attention List, when she looks at waitlist demand for a day, then she sees only an aggregate count, never individual client identities or entries.

**FEAT-20.SPEC-003-AC-12:** Given Support opens FEAT-19's read-only view of a Pro's account, when they inspect waitlist activity, then they see entry state but never a control to edit or delete an entry.

**FEAT-20.SPEC-003-AC-13:** Given Riley's requested Service is archived between screen load and submit, when she submits, then she sees "This service is no longer available. Choose another service." and no entry is created.

**FEAT-20.SPEC-003-AC-14:** Given the Pro's booking horizon shortens between Riley starting the form and submitting it, when she submits a range that was valid at load but is no longer valid at submit, then it is rejected against the current, not the original, horizon.

**FEAT-20.SPEC-003-AC-15:** Given a single-day submission is malformed with unequal start_date and end_date, when validation runs, then it is rejected by the date-range-shape rule.

**FEAT-20.SPEC-003-AC-16:** Given Riley attempts to edit an existing entry's service or date range after creation, when she looks for a way to do so, then no edit control exists anywhere in the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |
