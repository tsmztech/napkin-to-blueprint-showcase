---
document_type: spec
spec_type: automation
spec_id: FEAT-29.SPEC-008
spec_name: Account Closure Orchestration
spec_slug: account-closure-orchestration
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Account Closure Orchestration

## Overview

**Name:** Account Closure Orchestration
**ID:** FEAT-29.SPEC-008
**Type:** Automation
**Purpose:** Sequences closure -- cancels the subscription, takes the booking page down, starts the 30-day cooling-off clock, and executes deletion (retaining only legally required de-identified financial records) when the cooling-off period expires unreversed.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Sequencing the closure steps once the Pro confirms closure with no upcoming bookings remaining
- Starting and tracking the cooling-off clock
- Executing permanent deletion when the cooling-off period expires unreversed
- Sending the closure confirmation and, later, the deletion notice

**Non-Goals:**
- Cancelling upcoming bookings with full refunds -- owned by FEAT-30 (Pro Booking Management), invoked by FEAT-29.SPEC-005 before this automation ever starts; this automation assumes no upcoming bookings remain
- Reversing closure during the cooling-off period -- owned by FEAT-29.SPEC-009 (Account Reopening); this automation only starts and completes the one-way sequence, or stops if reopening intervenes
- Defining the cooling-off period length and retention scope -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules), which this automation implements
- Deleting the legally required de-identified financial history -- excluded per scope-boundaries.md SC-22: permanent deletion never removes the de-identified records the law requires to be retained

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro confirms account closure | FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Pro confirms "Close my account" with zero upcoming bookings remaining | Pro Account reference |
| Cooling-off period expires | System (scheduled check against Pro Account.status = Closing and its closure request date) | The account has been in Closing status for platform parameter: `account-closure-cooling-off-days` with no reopening | Pro Account reference, closure request date |

## Processing Logic

**Closure-start path:**
1. Receive the closure confirmation with the Pro Account reference.
2. Verify no upcoming bookings remain for this Pro Account (a final safety check; FEAT-29.SPEC-005 has already ensured this through FEAT-30).
3. Cancel the Subscription via FEAT-18.SPEC-006 (subscription-billing capability).
4. Take the booking page down (Pro Account status change signals FEAT-05.SPEC-008 to render "this booking page isn't available").
5. Set Pro Account.status to Closing and record the closure request date.
6. Trigger FEAT-29.SPEC-017 (Account Closure & Deletion Notifications) to send the closure confirmation.

**Deletion path (cooling-off expiry):**
1. On the scheduled check, find every Pro Account whose status is Closing and whose closure request date is at least platform parameter: `account-closure-cooling-off-days` in the past, with no reopening having occurred.
2. For each such account: hard-delete the Pro's personal data -- sign_in_email, sign_in_mobile, signed_in_devices, and profile fields (display_name, photo, intro, studio_address, and other Pro Account fields).
3. Hard-delete Client contact details and notes for every Client belonging to this Pro Account, following the same semantics as XBR-19 (FEAT-13's client deletion).
4. De-identify Activity Events belonging to this Pro Account per FEAT-16.SPEC-005.
5. Retain Booking and Deposit Transaction history in de-identified form only, as legally required (SC-22) -- no further fields beyond what the law requires are kept.
6. Set Pro Account.status to Closed.
7. Trigger FEAT-29.SPEC-017 to send the final deletion notice.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Closure started | Confirmation received with zero upcoming bookings | Subscription cancelled; booking page taken down; Pro Account.status set to Closing; closure request date recorded | FEAT-29.SPEC-005 shows the Closing state with days remaining | FEAT-29.SPEC-005, FEAT-18, FEAT-05.SPEC-008, FEAT-29.SPEC-017 |
| Closure blocked | Upcoming bookings still exist at the moment of processing (a race with a newly created booking) | No status change | FEAT-29.SPEC-005 re-shows the updated upcoming-bookings list and requires the cancel-and-continue step again | FEAT-29.SPEC-005 |
| Permanent deletion executed | Cooling-off period expires with no reopening | Personal data hard-deleted; Client contact details hard-deleted; Activity Events de-identified; Booking/Deposit Transaction retained de-identified; Pro Account.status set to Closed | FEAT-29.SPEC-017 sends the final deletion notice; no in-product screen remains to show this state to the (now-deleted) account | FEAT-29.SPEC-017, FEAT-16.SPEC-005, FEAT-13 |
| Deletion skipped (reopened) | Reopening (FEAT-29.SPEC-009) occurred before the scheduled deletion check runs | No deletion occurs for this account | -- | FEAT-29.SPEC-009 |
| Closure-start failure | The subscription-cancel or booking-page-takedown step fails | No status change; the account remains Active/Paused | FEAT-29.SPEC-005 shows "Couldn't close your account. Try again." with a retry action | FEAT-29.SPEC-005 |

## Data Model

**Reads:** Pro Account (status, closure request date); Booking (to verify no upcoming bookings remain); Subscription (to cancel).
**Creates:** None.
**Updates:** Pro Account.status (Active/Paused -> Closing -> Closed); Subscription.status (-> Cancelled via FEAT-18.SPEC-006).
**Deletes:** Pro Account's personal fields (sign_in_email, sign_in_mobile, signed_in_devices, profile fields) on permanent deletion; Client contact details and notes for every Client of this Pro Account on permanent deletion. Booking and Deposit Transaction records are never deleted -- only de-identified.

## Business Rules

- XBR-20: upcoming bookings must first be cancelled with full refunds; the subscription is cancelled and the booking page taken down before the cooling-off clock starts; data is deleted only after the cooling-off period, keeping only legally required de-identified financial records.
- The cooling-off period is fixed at platform parameter: `account-closure-cooling-off-days` (FEAT-29.SPEC-013) -- this automation never shortens or extends it per account.
- Deletion never removes the de-identified financial records the law requires to be retained (SC-22) -- there is no "delete everything, no exceptions" option.
- Client contact-detail deletion on account closure follows the same hard-delete-with-de-identified-history semantics as XBR-19 (FEAT-13).
- Reopening (FEAT-29.SPEC-009) at any point before the scheduled deletion check runs cancels the deletion outcome entirely for that account -- there is no partial or in-progress deletion state that reopening must unwind.

## Edge Cases

- **A booking is created for this Pro Account between FEAT-29.SPEC-005's review step and this automation's own final safety check** -- Closure is blocked and the Pro is returned to the review step to clear the new booking; no partial closure (e.g., subscription cancelled but booking page still live) is ever left in place.
- **The subscription-cancel step succeeds but the booking-page-takedown step fails** -- The entire closure-start sequence is treated as a single unit: if any step fails, the automation reports failure and Pro Account.status is not advanced to Closing, so the account remains fully Active/Paused rather than left in a partially-closed state. A retry re-attempts the full sequence from the top, including re-invoking FEAT-18.SPEC-006's subscription cancellation; that call is idempotent against a subscription already in Cancelled status (cancelling an already-cancelled subscription is a no-op that leaves it Cancelled and returns success), so the retry never double-cancels or errors on the already-completed step -- it simply proceeds to re-attempt the booking-page takedown that failed.
- **Concurrent trigger firing -- the Pro confirms closure from two devices at nearly the same time** -- Only the first confirmation to commit transitions the account to Closing; the second is a no-op against an account already in Closing (idempotent), and its screen reflects the Closing state on next load.
- **Trigger fires while a previous run is in flight -- the scheduled deletion check runs while a closure-start is still processing for the same account** -- The deletion check only ever considers accounts already in Closing status with a recorded closure request date; an account whose closure-start has not yet committed that status cannot be selected by the same run, so no conflict arises.
- **The Pro reopens the account in the same window the scheduled deletion check is evaluating it** -- Reopening (FEAT-29.SPEC-009) is the authoritative status change; if it commits before the deletion check reads the account's status, the account is no longer Closing and is excluded from that run. If the deletion check has already begun processing that specific account in the same run, reopening is refused with the message defined in FEAT-29.SPEC-009's own Edge Cases, since deletion for that account is by then irreversible.
- **A refund tied to this Pro Account's bulk cancellation (FEAT-30) is still "in progress" when the cooling-off clock starts** -- The cooling-off clock and the refund's own completion are independent; the refund continues per its own retry rules (XBR-10) regardless of the account's Closing status, and its resulting Deposit Transaction record is retained (de-identified, if the account reaches permanent deletion) exactly like any other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Triggered by (inbound) | Confirmation of closure starts this automation |
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Affects (outbound) | Shows the resulting Closing state or failure |
| FEAT-18 (Pro Subscription Billing & Account Management) | Affects (outbound) | Subscription cancellation via FEAT-18.SPEC-006 |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | Affects (outbound) | Booking page takedown |
| FEAT-29.SPEC-009 (Account Reopening) | References (inbound) | Reopening cancels this automation's deletion outcome |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Cooling-off period and retention scope |
| FEAT-29.SPEC-017 (Account Closure & Deletion Notifications) | Affects (outbound) | Closure confirmation and final deletion notice |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | Affects (outbound) | De-identification of Activity Events on deletion |
| FEAT-13 (Client Record Management) | Affects (outbound) | Client contact-detail deletion, following XBR-19 semantics |

## Analytics and Success Signals

- **account_closure_requested** (had_upcoming_bookings: yes/no) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list
- **account_deleted** (-- no properties beyond the event itself, since the account no longer exists to attach further context to) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so permanent deletion remains observable for operational and compliance monitoring

## Acceptance Criteria

**FEAT-29.SPEC-008-AC-01:** Given Talia confirms closure with zero upcoming bookings, when this automation runs, then her Subscription is cancelled, her booking page is taken down, and Pro Account.status is set to Closing with today's date recorded.

**FEAT-29.SPEC-008-AC-02:** Given Talia's closure-start commits successfully, when the sequence completes, then FEAT-29.SPEC-017 sends the closure confirmation.

**FEAT-29.SPEC-008-AC-03:** Given a new booking appears for Talia's account between screen review and this automation's final check, when the automation runs, then closure is blocked and no status change occurs.

**FEAT-29.SPEC-008-AC-04:** Given the booking-page-takedown step fails during closure-start, when the failure is detected, then Pro Account.status is not advanced to Closing and the account remains fully Active/Paused.

**FEAT-29.SPEC-008-AC-05:** Given Talia's account has been Closing for exactly platform parameter: `account-closure-cooling-off-days` with no reopening, when the scheduled deletion check runs, then her personal data and her clients' contact details are hard-deleted, her Activity Events are de-identified, and Pro Account.status is set to Closed.

**FEAT-29.SPEC-008-AC-06:** Given Talia's account reaches permanent deletion, when the deletion completes, then her Booking and Deposit Transaction history is retained in de-identified form only, per SC-22.

**FEAT-29.SPEC-008-AC-07:** Given Talia's account reaches permanent deletion, when FEAT-29.SPEC-009 (Account Reopening) is attempted afterward, then no reopening path exists -- deletion is irreversible.

**FEAT-29.SPEC-008-AC-08:** Given Talia reopens her account (FEAT-29.SPEC-009) before the scheduled deletion check runs, when the check next runs, then her account is excluded from deletion because its status is no longer Closing.

**FEAT-29.SPEC-008-AC-09:** Given Talia confirms closure from two devices at nearly the same time, when both confirmations process, then only the first commits the Closing transition and the second is a no-op against an account already Closing.

**FEAT-29.SPEC-008-AC-10:** Given a goodwill refund from Talia's bulk cancellation is still "in progress" when her cooling-off clock starts, when the refund's own retry cycle completes, then it proceeds independently of the account's Closing status per XBR-10.

**FEAT-29.SPEC-008-AC-11:** Given Talia's closure-start sequence fails partway (subscription cancelled, booking-page takedown fails), when the automation reports the failure, then FEAT-29.SPEC-005 shows "Couldn't close your account. Try again." and no partially-closed state is left visible.

**FEAT-29.SPEC-008-AC-12:** Given the scheduled deletion check has already begun processing Talia's account in the current run, when a reopening attempt arrives in that same window, then it is refused per FEAT-29.SPEC-009's own edge-case handling, since deletion is by then irreversible.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
