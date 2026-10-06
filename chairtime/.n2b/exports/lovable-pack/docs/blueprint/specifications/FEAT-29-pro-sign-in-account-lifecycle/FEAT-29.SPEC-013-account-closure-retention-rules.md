---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-29.SPEC-013
spec_name: Account Closure & Retention Rules
spec_slug: account-closure-retention-rules
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 19
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Account Closure & Retention Rules

## Overview

**Name:** Account Closure & Retention Rules
**ID:** FEAT-29.SPEC-013
**Type:** Logic/Rule
**Purpose:** Governs the 30-day cooling-off period, the closure sequencing required before deletion, the scope of what deletion removes versus retains, and the scope of the data export.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle
**Governed Entity:** Pro Account (status/lifecycle slice), and by reference the Client, Booking, and Deposit Transaction entities as they are affected by closure, deletion, and export

## Scope and Non-Goals

**In Scope:**
- The cooling-off period length and what it protects
- The mandated sequencing of closure steps (bookings cancelled and refunded, subscription cancelled, booking page down, then the cooling-off clock starts)
- What permanent deletion removes and what it retains
- The scope of the data export (which fields, which entities, and the explicit private_note exclusion)
- Authorization for closure, reopening, and export actions

**Non-Goals:**
- Sign-in code and session rules -- owned by FEAT-29.SPEC-011; this spec governs account lifecycle and data retention, not sign-in itself
- Contact-change confirmation -- owned by FEAT-29.SPEC-012
- Executing the bulk cancellation, subscription cancellation, or booking-page takedown -- owned by FEAT-30, FEAT-18, and FEAT-05.SPEC-008 respectively; this spec states the sequencing rule that FEAT-29.SPEC-008 orchestrates against
- Deleting the legally required de-identified financial history -- excluded per scope-boundaries.md SC-22: this spec explicitly carves out that retention as a legal requirement, not a discretionary choice

## Governed Entity

**Entity:** Pro Account (status/lifecycle slice)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| status | enum (Active \| Paused \| Closing \| Closed) | This feature owns only the Closing/Closed/reopened-to-Active slice of this shared field |
| closure_request_date (derived) | date | The date closure was confirmed, used to compute cooling-off expiry |

**Referenced entities (by reference, not owned by this spec):** Client (contact details, notes -- deletion scope), Booking and Deposit Transaction (retention scope after deletion), Activity Event (de-identification scope after deletion).

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-29.SPEC-004 | Data Export Screen | Displays the export-scope explanation to the Pro |
| FEAT-29.SPEC-005 | Account Closure & Reopening Screen | Displays the cooling-off period and closure sequencing to the Pro |
| FEAT-29.SPEC-007 | Data Export Generation | Enforces the export scope when assembling the file |
| FEAT-29.SPEC-008 | Account Closure Orchestration | Enforces closure sequencing, cooling-off timing, and deletion scope |
| FEAT-29.SPEC-009 | Account Reopening | Enforces the cutoff after which reopening is no longer possible |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status | Transition to Closing is only valid when zero upcoming Bookings remain for this Pro Account | On closure confirmation | On closure confirmation (FEAT-29.SPEC-008) | -- (the screen re-shows the upcoming-bookings review rather than an error message; see FEAT-29.SPEC-005) | Yes |
| status | Transition from Closing to Active (reopening) is only valid before the scheduled deletion check has begun processing the account | On reopening request | On reopening request (FEAT-29.SPEC-009) | "This account can no longer be reopened." | Yes |
| closure_request_date | Set once, at the moment status transitions to Closing; cleared on reopening | Always | On closure confirmation and on reopening | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Cooling-off expiry | status, closure_request_date | Deletion is eligible only when status is Closing and the current date is at least platform parameter: `account-closure-cooling-off-days` past closure_request_date | -- |
| Closure sequencing | status, Subscription.status, Booking Page availability | The account cannot transition to Closing until upcoming bookings are cleared (cancelled and refunded); Subscription cancellation and booking-page takedown occur as part of the same closure-start step, before the cooling-off clock starts (XBR-20) | -- |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|-----------------------------------------------|
| Request account closure | The Pro (Talia) | Only for their own account; only once zero upcoming bookings remain | -- |
| Request account closure | Platform Operator (Support) | Never (SC-05) | No closure control is rendered on Support's view |
| Reopen the account | The Pro (Talia) | Only for their own account, only while status is Closing and deletion has not yet begun processing it | "This account can no longer be reopened." once deletion has begun |
| Reopen the account | Platform Operator (Support) | Never | No reopening control is rendered on Support's view |
| Request a data export | The Pro (Talia) | Only for their own account's own data | -- |
| Request a data export | Platform Operator (Support) | Never -- support never sees or triggers a Pro's export | No export control is rendered on Support's view |
| View account status (Active/Paused/Closing/Closed) | Platform Operator (Support) | Always, view-only, after a Pro's help request (XBR-24) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| closure_request_date | Current date | On closure confirmation | No |
| Deletion eligibility date | closure_request_date + platform parameter: `account-closure-cooling-off-days` | Whenever status is Closing | No |

## Business Rules

- XBR-20 (account closure): upcoming bookings must first be cancelled with full refunds; the subscription is cancelled and the booking page taken down; data is deleted after a platform parameter: `account-closure-cooling-off-days` cooling-off period, keeping only legally required de-identified financial records.
- Permanent deletion hard-deletes: the Pro's sign-in email/mobile, signed-in devices, and profile fields; and, for every Client of this Pro Account, contact details and notes (following XBR-19 semantics).
- Permanent deletion never removes: legally required de-identified financial records (Booking and Deposit Transaction history, retained de-identified per SC-22); Activity Events are de-identified rather than deleted (FEAT-16.SPEC-005), preserving dispute-evidence integrity without retaining identifiable personal data.
- The data export's scope is exactly: the structured fields of Client (name, phone, email, booking_history reference), Booking (service, timing, price, deposit, state, timestamps), and Deposit Transaction (amount, status, outcome, timestamps) for the requesting Pro's own account only -- explicitly excluding Client.private_note (Coordination Note 4: private_note is Pro-only working notes, not a client or booking record proper).
- A closure or export action is never available to any role but the Pro whose account it concerns; Support's role throughout this spec is view-only status, per XBR-24.

## Edge Cases

- **Deletion eligibility date falls exactly on the current date** -- Eligible; the rule is "at least" the cooling-off period, so a request evaluated on the exact boundary day qualifies for deletion processing.
- **The Pro requests an export the same day the cooling-off period is about to expire** -- The export is still generated normally per its own scope rule; requesting an export has no effect on the cooling-off clock or deletion eligibility, since the two are independent actions.
- **A Client record has an upcoming booking at the moment of a de-identification pass triggered by this Pro's account deletion** -- Cannot occur under normal sequencing, since XBR-20 requires all upcoming bookings to be cancelled and refunded before the cooling-off clock can even start; if a data inconsistency were somehow found, deletion processing would not proceed for that Client until the inconsistency is resolved, consistent with FEAT-13's own deletion-eligibility rule (XBR-19: deletion is refused while an upcoming booking exists).
- **Two competing intentions -- the Pro requests reopening and, in the same visit, immediately requests a fresh export** -- Independent actions; reopening changes status to Active, and an export request that follows reads the now-Active account's current data normally.
- **The de-identification pass for Activity Events runs concurrently with the hard-delete pass for Client contact details** -- Both are part of the same deletion sequence (FEAT-29.SPEC-008) and operate on disjoint fields (Activity Event content vs. Client contact fields), so no field-level conflict exists between them; the sequence completes both before setting Pro Account.status to Closed.
- **A legally required de-identified financial record is later needed for a dispute after the Pro's account is deleted** -- It remains available in de-identified form indefinitely (SC-22); no further purge ever removes it, since this spec explicitly carves it out of every deletion pass.

## Acceptance Criteria

**FEAT-29.SPEC-013-AC-01:** Given Talia's account has zero upcoming bookings, when she confirms closure, then status transitions to Closing and closure_request_date is set to today.

**FEAT-29.SPEC-013-AC-02:** Given Talia's account has upcoming bookings, when she attempts to confirm closure without clearing them, then the transition to Closing is refused and the upcoming-bookings review is shown instead.

**FEAT-29.SPEC-013-AC-03:** Given Talia's account has been Closing for exactly platform parameter: `account-closure-cooling-off-days`, when the deletion eligibility check runs, then the account is eligible for deletion.

**FEAT-29.SPEC-013-AC-04:** Given Talia's account has been Closing for one day fewer than platform parameter: `account-closure-cooling-off-days`, when the deletion eligibility check runs, then the account is not yet eligible.

**FEAT-29.SPEC-013-AC-05:** Given Talia's account reaches permanent deletion, when the deletion executes, then her sign-in email/mobile, signed-in devices, and profile fields are hard-deleted, along with her clients' contact details and notes.

**FEAT-29.SPEC-013-AC-06:** Given Talia's account reaches permanent deletion, when the deletion executes, then her Booking and Deposit Transaction history is retained in de-identified form only, and her Activity Events are de-identified rather than deleted.

**FEAT-29.SPEC-013-AC-07:** Given Talia requests a data export, when the file is generated, then it includes only structured Client, Booking, and Deposit Transaction fields for her own account, excluding Client.private_note.

**FEAT-29.SPEC-013-AC-08:** Given deletion has already begun processing Talia's account, when she attempts to reopen it, then the request is refused with "This account can no longer be reopened."

**FEAT-29.SPEC-013-AC-09:** Given Platform Operator (Support) views a Pro's account, when they look for a closure, reopening, or export control, then none is rendered anywhere on their view.

**FEAT-29.SPEC-013-AC-10:** Given Talia requests an export on the same day her cooling-off period is set to expire, when the export generates, then it completes normally with no effect on the deletion eligibility clock.

**FEAT-29.SPEC-013-AC-11:** Given Talia's account has already been permanently deleted, when a card-issuer dispute later requires her historical financial records, then the de-identified Booking and Deposit Transaction history remains available.

**FEAT-29.SPEC-013-AC-12:** Given the deletion sequence for Talia's account runs, when both the Activity Event de-identification pass and the Client contact-detail hard-delete pass execute, then both complete before Pro Account.status is set to Closed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
