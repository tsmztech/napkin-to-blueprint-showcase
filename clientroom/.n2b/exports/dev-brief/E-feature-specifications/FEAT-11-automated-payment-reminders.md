# FEAT-11 — Automated Payment Reminders

This chapter covers Automated Payment Reminders, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 5 specifications carrying 66 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-11.SPEC-001 | Reminder Schedule | automation | 12 |
| FEAT-11.SPEC-002 | Reminder Eligibility Rule | logic-rule | 16 |
| FEAT-11.SPEC-003 | Invoice Reminder Panel | screen | 16 |
| FEAT-11.SPEC-004 | Overdue Reminder Email | notification | 12 |
| FEAT-11.SPEC-005 | Bank-Transfer-Pending Reminder Pause | automation | 10 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Automated Payment Reminders

## Summary

**Feature:** Automated Payment Reminders
**ID:** FEAT-11
**Description:** An overdue invoice automatically triggers polite reminder emails on day 3 and day 10 overdue, with no manual action from the freelancer, and stops the instant the invoice is paid.
**Priority:** Core
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** BRIEF.md, Experience narrative: "When an invoice goes overdue, polite reminders go out on day 3 and day 10 without you typing a word" — this is the brief's explicit answer to "the freelancer stops chasing" (BRIEF.md, Vision). MVP phase: central to the "get paid faster, chase less" promise. [RESEARCH-INFORMED: automated invoice reminders are standard in HoneyBook and Bonsai and described by Moxie and SuiteDash; Moxie users report automated emails that could not be stopped once triggered (Reddit via independent reviews, MEDIUM), which is why per-invoice pause and automatic stop on payment are part of this feature] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Automatic reminders — day-3 and day-10 overdue emails with no manual trigger
- Pause per invoice — freelancer can pause reminders for a specific invoice (e.g., a dispute in progress)
- Automatic stop — reminders stop the moment the invoice is paid
- Send a manual reminder — one click sends a polite reminder at any time after the due date, including after the day-10 reminder [AUDIT-ADDED: 1 -- journey walk: the draft left no next step once both automatic reminders had gone out]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-11.SPEC-001 | Reminder Schedule | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | Automatically sends the day-3 and day-10 overdue reminder for an unpaid invoice, re-checking eligibility immediately before each send |
| FEAT-11.SPEC-002 | Reminder Eligibility Rule | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact) | Governs whether a reminder (automatic or manual) is allowed to send: paid/pause/pending checks, time-zone day counting, and the one-manual-reminder-per-day limit |
| FEAT-11.SPEC-003 | Invoice Reminder Panel | Screen | Nadia (Freelancer), Dana (Support Operator) | Shows an invoice's reminder history and lets Nadia pause/resume the schedule or send a manual reminder |
| FEAT-11.SPEC-004 | Overdue Reminder Email | Notification | Owen (Client Primary Contact), Dana (Support Operator) | The polite overdue-payment email sent to the Primary Contact for a day-3, day-10, or manual reminder |
| FEAT-11.SPEC-005 | Bank-Transfer-Pending Reminder Pause | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | Automatically pauses and resumes an invoice's reminder schedule while a bank-transfer payment is pending (inbound from FEAT-10) |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Automatic reminders | FEAT-11.SPEC-001, FEAT-11.SPEC-004 | Reminder Schedule fires the day-3/day-10 send; Overdue Reminder Email is the message content and delivery behavior | Phase 2 (Explicit) |
| Pause per invoice | FEAT-11.SPEC-003 | Pause/resume control on the Invoice Reminder Panel, scoped to one invoice | Phase 2 (Explicit) |
| Automatic stop | FEAT-11.SPEC-002 | Eligibility rule re-checked immediately before every send; a Paid invoice fails the check and no further reminder sends | Phase 2 (Explicit) |
| Send a manual reminder | FEAT-11.SPEC-003, FEAT-11.SPEC-004 | One-click action on the panel, gated by the eligibility rule's rate limit, sends the same Overdue Reminder Email | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-11.SPEC-005 | Bank-Transfer-Pending Reminder Pause | Phase 4 (Trigger-Response / External Dependencies lens) | The Validation & Limits field states "Reminders also pause automatically while a bank-transfer payment is pending (FEAT-10)"; this is a cross-feature side-effect with its own trigger, resolution, and re-resolution behavior, so it rises to a standalone Automation spec per the Phase 4 disposition rule rather than staying an annotation on SPEC-001 |

## Entity-Lifecycle Coverage Matrix

**Entity: Reminder Log**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-11.SPEC-001, FEAT-11.SPEC-003 | Reminder Schedule creates a `day 3` / `day 10` entry on each automatic send; the manual-send action on the panel creates a `manual` entry | -- |
| Read (single) | FEAT-11.SPEC-003 | Invoice Reminder Panel shows one invoice's reminder history | -- |
| Read (list) | FEAT-11.SPEC-003 | Same panel lists every entry for the invoice in send order | This feature has no cross-invoice reminder list; the invoice detail is the only surface |
| Update | FEAT-11.SPEC-003, FEAT-11.SPEC-005, FEAT-11.SPEC-001 | `pause_state` updated by Nadia's pause/resume action (SPEC-003) and by the automatic bank-transfer-pending trigger (SPEC-005); `sent_at`/status updated when the Reminder Schedule completes a send (SPEC-001) | Contention rule from the dependency map: the schedule re-checks pause/pending/paid state immediately before each send, and a pause saved first wins |
| Delete/Archive | N/A | Reminder Log deletion belongs to Delete My Account & Data (FEAT-24), subject to legal financial-record retention (dependency map, scope-boundaries.md SC-24); this feature never deletes or archives reminder history itself -- recorded as an explicit non-goal | -- |
| State Transition | FEAT-11.SPEC-003, FEAT-11.SPEC-005 | `pause_state` transitions: Active -> Paused by freelancer (and back) via SPEC-003; Active -> Paused while bank transfer pending (and back) via SPEC-005 | The two pause reasons are independent triggers on the same field; SPEC-002's eligibility check treats either as "not eligible to send" |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Invoice | FEAT-11.SPEC-001, FEAT-11.SPEC-002, FEAT-11.SPEC-003 | Reads due date (from FEAT-09), current status (Paid, Payment pending, Overdue) to compute the reminder window and eligibility, and to display invoice context on the panel |
| Freelancer Account | FEAT-11.SPEC-002 | Reads time zone (FEAT-15) so day-3/day-10 counting follows the freelancer's local calendar day, not a fixed offset |
| Client Contact | FEAT-11.SPEC-004 | Reads the invoice's Primary Contact as the reminder's recipient |

**Flagged discrepancy (not resolved by this Brief):** the feature's Connected Entities line in product-features.md lists Invoice as read-only for this feature, but the dependency map's Invoice field list attributes the `reminder_paused` field and the Overdue status flag to FEAT-11 as an updater. This Brief treats `reminder_paused` as written by FEAT-11.SPEC-003's pause/resume action (consistent with the Key Capability "Pause per invoice") and treats "Overdue" as a status derived by FEAT-11.SPEC-002 from FEAT-09's due date rather than a field this feature writes independently of that derivation. The inconsistency between "Invoice (read)" and the dependency map's updater list is flagged here per the methodology's instruction to flag, not resolve, Interactions discrepancies; the Requirements Architect should reconcile the Connected Entities line on the next Stage 2 pass.

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Invoice due date passes unpaid (day 3, freelancer's time zone) | Send the day-3 reminder email | Standalone Automation + Standalone Notification | FEAT-11.SPEC-001 / FEAT-11.SPEC-004 |
| Invoice still unpaid at day 10 | Send the day-10 reminder email | Standalone Automation + Standalone Notification | FEAT-11.SPEC-001 / FEAT-11.SPEC-004 |
| A scheduled reminder send fails | Retry automatically; log the failure rather than dropping it silently | Inline in triggering automation | FEAT-11.SPEC-001 |
| Invoice is marked Paid (by Owen, by Nadia recording an off-platform payment, or by processor confirmation) | Any remaining scheduled reminder is blocked at its next eligibility check | Standalone Logic/Rule | FEAT-11.SPEC-002 |
| Nadia pauses reminders for one invoice | `pause_state` -> Paused by freelancer; schedule skipped until resumed | Standalone Screen action | FEAT-11.SPEC-003 |
| Nadia resumes reminders for one invoice | `pause_state` -> Active; schedule resumes at its normal day-3/day-10 points | Inline in triggering screen | FEAT-11.SPEC-003 |
| Invoice enters Payment pending (bank transfer, FEAT-10) | `pause_state` -> Paused while bank transfer pending | Standalone Automation (cross-feature) | FEAT-11.SPEC-005 |
| Payment pending resolves or reverts (FEAT-10) | `pause_state` -> Active (unless still freelancer-paused) | Standalone Automation (cross-feature) | FEAT-11.SPEC-005 |
| Nadia sends a manual reminder | One-per-day limit enforced; reminder email sent; entry logged | Standalone Notification, rate limit owned by Logic/Rule | FEAT-11.SPEC-003 (action) / FEAT-11.SPEC-002 (limit) / FEAT-11.SPEC-004 (email) |
| Any reminder sends (automatic or manual) | An append-only Activity Log Entry is written (actor: the product; event: reminder sent) | Cross-feature -- logged in touchpoints | FEAT-13 responsibility (XBR-05) |
| Owen clicks the pay link inside a reminder email | Navigates to the payment flow for that invoice | Cross-feature -- logged in touchpoints | FEAT-10 responsibility |

## Shared Context

**Shared Entities:**
- Reminder Log -- created by SPEC-001 (automatic sends) and SPEC-003 (manual send); read/displayed by SPEC-003; updated (`pause_state`) by SPEC-003 and SPEC-005; updated (`sent_at`/status) by SPEC-001. Fields: invoice, reminder_type (day 3 / day 10 / manual), scheduled_for, sent_at, pause_state (Active / Paused by freelancer / Paused while bank transfer pending).
- Invoice (referenced) -- due date and status read by SPEC-001, SPEC-002, and SPEC-003; see the flagged Connected-Entities discrepancy above regarding `reminder_paused` and the Overdue flag.

**Shared UI Patterns:**
- N/A -- this feature produces a single Screen spec (the Invoice Reminder Panel), so no pattern is shared across multiple screens within the feature. The panel itself is expected to sit inside the Invoice Detail screen owned by Invoice Generation & Sending (FEAT-09); Spec Writers for both should keep the panel's placement and reminder-history layout consistent with that host screen.

**Shared Validation:**
- FEAT-11.SPEC-002 (Reminder Eligibility Rule) is the single source of truth for "may a reminder send right now": paid check, freelancer-pause check, bank-transfer-pending check, and the manual-reminder one-per-day limit. FEAT-11.SPEC-001, FEAT-11.SPEC-003, and FEAT-11.SPEC-005 all reference SPEC-002 rather than re-implementing any of these checks.

## Internal Dependency Map

```
SPEC-001 (Reminder Schedule) -> [day 3 / day 10 elapses in the freelancer's time zone] -> SPEC-002 (Reminder Eligibility Rule) -> [eligible] -> SPEC-004 (Overdue Reminder Email)
SPEC-002 (Reminder Eligibility Rule) -> [not eligible: paid, paused, or pending] -> SPEC-001 (send skipped, no further automatic reminders after day 10)
SPEC-003 (Invoice Reminder Panel) -> [Nadia taps Pause] -> Reminder Log pause_state updated -> [next scheduled check reads] -> SPEC-002
SPEC-003 (Invoice Reminder Panel) -> [Nadia taps Resume] -> Reminder Log pause_state updated -> [next scheduled check reads] -> SPEC-002
SPEC-003 (Invoice Reminder Panel) -> [Nadia taps Send Reminder Now] -> SPEC-002 (rate-limit check) -> [allowed] -> SPEC-004 (Overdue Reminder Email)
SPEC-005 (Bank-Transfer-Pending Reminder Pause) -> [FEAT-10 marks Payment pending] -> Reminder Log pause_state updated -> [read by] -> SPEC-002
SPEC-005 (Bank-Transfer-Pending Reminder Pause) -> [FEAT-10 pending resolves/reverts] -> Reminder Log pause_state updated -> [read by] -> SPEC-002
SPEC-001 (Reminder Schedule) -> [displays resulting history] -> SPEC-003 (Invoice Reminder Panel)
```

**Default Entry:** SPEC-003 (Invoice Reminder Panel) -- this feature has no standalone landing screen; Nadia reaches it from an overdue invoice's detail view (opened from the dashboard, FEAT-12), and Owen never sees it directly (he only receives SPEC-004's emails).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-11.SPEC-001 | Inbound | FEAT-09 (Invoice Generation & Sending) | Reads the invoice's due date to compute the day-3/day-10 schedule | Invoice is sent and its due date is set |
| FEAT-11.SPEC-002 | Inbound | FEAT-10 (Invoice Payment Processing) | Eligibility check reads the invoice's paid status to stop further reminders | Invoice is marked Paid (by Owen, Nadia, or processor confirmation) |
| FEAT-11.SPEC-005 | Inbound | FEAT-10 (Invoice Payment Processing) | Auto-pauses and auto-resumes the reminder schedule around a pending bank-transfer payment | Invoice enters or leaves Payment pending status |
| FEAT-11.SPEC-002 | Inbound | FEAT-15 (Currency, Tax, Time Zone & Locale) | Day-3/day-10 counting uses the freelancer's stored time zone | Reminder Schedule evaluates the due-date offset |
| FEAT-11.SPEC-001 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Writes a trail entry for each automatic reminder sent | Reminder Schedule completes a send |
| FEAT-11.SPEC-003 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Writes a trail entry for pause, resume, and manual-send actions | Nadia pauses, resumes, or manually sends a reminder |
| FEAT-11.SPEC-004 | Outbound | FEAT-14 (Notifications (Email)) | Uses the transactional email delivery capability to send and track the reminder email | Any reminder (automatic or manual) fires |
| FEAT-11.SPEC-004 | Outbound | FEAT-10 (Invoice Payment Processing) | The reminder email's pay link routes to the payment flow | Owen clicks the pay link in a reminder email |
| FEAT-11.SPEC-003 | Inbound | FEAT-12 (Freelancer Financial Dashboard) | Nadia opens an overdue invoice's reminder history/pause controls | Nadia clicks an Overdue invoice on her dashboard |
| FEAT-11.SPEC-003 | Outbound | FEAT-31 (Support Access & Session Logging) | Dana's read-only support session can view (never change) this panel | Dana opens a logged support session touching this invoice |

## Non-Functional Notes

**Data volumes / growth:** Each overdue invoice produces at most two automatic Reminder Log entries plus occasional manual entries (rate-limited to one per invoice per day); at the product's expected scale of a few thousand freelancers with 3-15 active clients each (scope-boundaries.md, SC-21), reminder volume per freelancer stays small and does not require special handling beyond ordinary list display.

**Responsiveness:** Automatic sending is a scheduled background behavior with no interactive loading state to manage (product-features.md, States); the Invoice Reminder Panel's history and pause/resume controls respond immediately to Nadia's actions, consistent with the product's general expectation that everyday actions complete without a perceptible wait.

**Data sensitivity / privacy:** Reminder Log data itself is low-sensitivity (send timestamps and pause state), but it is linked to the recipient Client Contact's identity, which is personal data under GDPR-class handling (assumptions-constraints.md, ASMP-24); the reminder panel and its history are hidden from Priya (Reviewer), who is never addressed by billing reminders, and are read-only for Dana inside a logged support session (user-persona.md, Access Matrix).

**Compliance flags:** ASMP-24 (GDPR-class personal data) applies to the recipient identity carried on every Reminder Log entry and Overdue Reminder Email. ASMP-26 (delivery reliability) directly shapes this feature's failure handling: a failed reminder send is retried automatically and surfaced rather than silently lost, feeding the Notification Delivery Reliability success metric. ASMP-25 (evidentiary correctness) applies indirectly through FEAT-13's immutable Activity Log Entry for every reminder sent, paused, resumed, or manually triggered, rather than through the mutable Reminder Log itself.

## Non-Goals

- **A configurable reminder cadence or automation builder** -- Excluded per scope-boundaries.md (SC-11): the product ships fixed, sensible behavior (day-3 and day-10 reminders) instead of a configurable workflow builder, in line with the brief's promise that the freelancer stops chasing without setup overhead.
- **Reminder channels beyond email (SMS, push, in-app-only alerts to Owen)** -- Adjacency exclusion: assumptions-constraints.md (ASMP-29) and BRIEF.md's Ecosystem & Integrations establish transactional email as the sole channel reaching client contacts ("clients will not install an app"); no Stage 2 document defines an SMS or push capability for this or any feature.
- **This feature deleting or purging Reminder Log entries** -- Intentional lifecycle decision surfaced by the CRUD matrix: Reminder Log deletion belongs entirely to Delete My Account & Data (FEAT-24), subject to legal financial-record retention (scope-boundaries.md, SC-24); this feature never removes reminder history on its own.
- **Priya (Client Reviewer Contact) as a reminder recipient or viewer** -- Grounded in user-persona.md's Access Matrix: Priya's Invoicing & Payments and billing-relevant Notifications access is None; reminders are addressed only to the invoice's Primary Contact, per the feature's Access field.



# Automation Spec: Reminder Schedule

## Overview

**Name:** Reminder Schedule
**ID:** FEAT-11.SPEC-001
**Type:** Automation
**Purpose:** Automatically sends the day-3 and day-10 overdue reminder email for an unpaid invoice, re-checking eligibility immediately before each send so a reminder never goes out to an invoice that has since been paid or paused.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders

## Scope and Non-Goals

**In Scope:**
- Detecting when an unpaid invoice's due date has reached day 3 and day 10, counted in the freelancer's time zone
- Re-checking eligibility (paid, freelancer-paused, bank-transfer-pending) immediately before every send, per FEAT-11.SPEC-002
- Creating the day-3 and day-10 Reminder Log entries and handing each off to the Overdue Reminder Email
- Retrying a failed send and logging the failure rather than dropping it silently
- Stopping automatic sends after day 10 -- no further automatic reminder exists for an invoice

**Non-Goals:**
- Determining *whether* a reminder is allowed to send right now (paid/pause/pending checks, rate limits) -- owned entirely by FEAT-11.SPEC-002 (Reminder Eligibility Rule); this automation calls that rule rather than re-implementing its checks.
- Nadia's manual "Send Reminder Now" action -- handled by FEAT-11.SPEC-003 (Invoice Reminder Panel); this automation covers only the two automatic (day-3, day-10) trigger points.
- Pausing or resuming the schedule around a pending bank-transfer payment -- a distinct cross-feature trigger owned by FEAT-11.SPEC-005 (Bank-Transfer-Pending Reminder Pause).
- A configurable reminder cadence, additional reminder points beyond day 3 and day 10, or a workflow builder -- excluded per scope-boundaries.md (SC-11): the product ships fixed, sensible behavior instead of a configurable automation builder.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An unpaid invoice's due date reaches 3 full elapsed days | System (schedule-based; anchored to the invoice's `due_date` from FEAT-09.SPEC-007 and evaluated against the freelancer's stored time zone per FEAT-15.SPEC-006) | Fires once per invoice, the first time the elapsed-day count (in the freelancer's local calendar day) reaches exactly 3, and only if no `day 3` Reminder Log entry already exists for this invoice | Invoice reference, `due_date`, current invoice status, freelancer's time zone, invoice amount/currency, the invoice's Primary Contact reference |
| An unpaid invoice's due date reaches 10 full elapsed days | System (schedule-based; same anchoring as the day-3 trigger) | Fires once per invoice, the first time the elapsed-day count reaches exactly 10, and only if no `day 10` Reminder Log entry already exists for this invoice | Same as above |

## Processing Logic

1. On each schedule evaluation, read every invoice whose current status is not Paid, Refunded, Partially refunded, or Corrected (i.e., an invoice that can still be overdue).
2. For each such invoice, compute the number of full calendar days elapsed since `due_date`, using the freelancer's stored time zone (FEAT-15.SPEC-006) rather than a fixed offset.
3. If the elapsed count is exactly 3 and no `day 3` Reminder Log entry exists for this invoice, mark this invoice as a day-3 candidate. If the elapsed count is exactly 10 and no `day 10` Reminder Log entry exists, mark it as a day-10 candidate. An invoice with neither condition produces no action.
4. For each candidate, immediately before sending, invoke the Reminder Eligibility Rule (FEAT-11.SPEC-002) against the invoice's current state (not the state read in step 1 -- the state at this exact moment).
5. If the eligibility check returns eligible: create a Reminder Log entry (`invoice`, `reminder_type` = `day 3` or `day 10`, `scheduled_for` = the computed due-date offset, `pause_state` = Active) and hand the entry off to the Overdue Reminder Email (FEAT-11.SPEC-004) for sending. On successful hand-off, set the entry's `sent_at` to the current time.
6. If the eligibility check returns not eligible (paid, paused by freelancer, or paused while a bank transfer is pending): take no action for this candidate. No Reminder Log entry is created for a skipped candidate -- there is nothing to log, since no reminder was scheduled or sent.
7. If the hand-off to the Overdue Reminder Email fails (the send itself fails, not the eligibility check), retry automatically; see Outcome Definitions for the failure path.
8. After the day-10 reminder has been sent (or skipped as ineligible), no further automatic trigger evaluates this invoice -- day 3 and day 10 are the only two automatic reminder points, per XBR-15.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Day-3 reminder sent | Elapsed days = 3, no prior day-3 entry, eligibility check passes | Reminder Log entry created with `reminder_type` = `day 3`, `sent_at` set | Reminder appears in the invoice's reminder history; Owen receives the email | FEAT-11.SPEC-003 (Invoice Reminder Panel), FEAT-11.SPEC-004 (Overdue Reminder Email), FEAT-13.SPEC-003 (Activity Entry Recording) |
| Day-10 reminder sent | Elapsed days = 10, no prior day-10 entry, eligibility check passes | Reminder Log entry created with `reminder_type` = `day 10`, `sent_at` set | Reminder appears in the invoice's reminder history; Owen receives the email | FEAT-11.SPEC-003, FEAT-11.SPEC-004, FEAT-13.SPEC-003 |
| Send skipped -- not eligible | Elapsed days = 3 or 10, but the invoice is Paid, or its `pause_state` is Paused by freelancer or Paused while bank transfer pending, at the eligibility re-check | No Reminder Log entry created | Nothing new appears in the reminder history; the invoice's current pause indicator (FEAT-11.SPEC-003) already reflects why | FEAT-11.SPEC-002 (Reminder Eligibility Rule), FEAT-11.SPEC-003 |
| No action -- day not yet reached | Elapsed days is not 3 and not 10 | None | None | -- |
| Send failed | Eligibility check passed but the hand-off to the Overdue Reminder Email fails (the email delivery capability reports a failure at send time, per FEAT-14.SPEC-001) | Reminder Log entry is created in a not-yet-sent state; retried per the retry rule below | No user-visible interruption; if retries exhaust, a delivery warning appears to Nadia on the affected project (XBR-30) | FEAT-11.SPEC-004, FEAT-13.SPEC-003 |

## Data Model

**Reads:** Invoice -- `due_date`, `status` (to determine candidacy and to feed the eligibility re-check, sourced from FEAT-09 and FEAT-10). Freelancer Account -- `time_zone` (FEAT-15), used to count elapsed days in local calendar days rather than a fixed offset. Reminder Log -- existing entries for the invoice, to enforce the once-per-threshold dedup check.
**Creates:** Reminder Log entry -- `invoice`, `reminder_type` (`day 3` or `day 10`), `scheduled_for`, `sent_at`, `pause_state` (copied from the invoice's current schedule state at creation).
**Updates:** Reminder Log entry -- `sent_at` and delivery status once the hand-off to FEAT-11.SPEC-004 completes or fails.
**Deletes:** None -- this automation never removes a Reminder Log entry (dependency map: Reminder Log deletion belongs to FEAT-24 only).

## Business Rules

- XBR-15: overdue reminders go out at day 3 and day 10 after the due date, counted in the freelancer's time zone; no automatic reminder exists past day 10.
- Eligibility is re-checked immediately before every send, not at the moment the day-3/day-10 threshold is first detected -- the two moments can differ if the schedule evaluation is delayed, and the invoice's state may have changed in between.
- A day-3 or day-10 Reminder Log entry is created only on an eligible send; a skipped candidate leaves no entry, since Reminder Log records reminders that were scheduled and sent, not reminders considered and declined.
- The dependency map's Contention note for Reminder Log applies: "a pause saved first wins" -- if Nadia's pause action (FEAT-11.SPEC-003) is saved before this automation's eligibility re-check runs for that invoice, the send is skipped; if the send has already completed before the pause is saved, the completed send stands.
- This automation is the sole owner of the day-3/day-10 timing decision; FEAT-11.SPEC-003's manual send and FEAT-11.SPEC-005's bank-transfer-pending pause never alter the day-3/day-10 dates themselves, only whether a given send is eligible.

## Edge Cases

- **Invoice is paid in the instant between candidate detection and the eligibility re-check** -- The eligibility re-check (step 4) reads the invoice's current state, catches the payment, and the send is skipped. This is the exact purpose of re-checking immediately before send rather than relying on the state read during candidate detection.
- **Due date falls on a time-zone transition (e.g., a daylight-saving change) in the freelancer's local calendar** -- Elapsed-day counting uses the freelancer's local calendar date boundaries as currently in effect; a transition day is still counted as exactly one elapsed day, consistent with how any other calendar day is counted.
- **The schedule evaluation itself is delayed or missed for a period (e.g., the automation was unavailable) and later catches up** -- On the next evaluation, an invoice already past day 3 or day 10 without a corresponding Reminder Log entry is still treated as a candidate for that threshold; a reminder that is now, in effect, being sent late is preferable to one silently skipped. It is not sent for both day 3 and day 10 simultaneously if both are overdue and unsent -- each threshold is evaluated and, if eligible, sent independently, in order (day 3 before day 10 in the same catch-up pass).
- **Invoice becomes ineligible for day 3 but later becomes eligible again before day 10** -- The day-3 opportunity is not retried once its window (exactly 3 elapsed days) has passed; only the day-10 trigger is still pending for that invoice, and it evaluates independently at day 10.
- **Concurrent trigger firing (two schedule evaluations run for the same invoice at effectively the same time)** -- The once-per-threshold dedup check (an existing Reminder Log entry for that `reminder_type`) is authoritative: whichever evaluation completes its eligibility check and entry creation first is the one that sends; the second evaluation's dedup check finds the just-created entry and takes no further action for that threshold.
- **A trigger fires for an invoice while a previous run (for the same invoice) is still in flight (send not yet confirmed)** -- The in-flight run holds the not-yet-sent Reminder Log entry for that threshold; a second concurrent run for the same threshold on the same invoice defers to the dedup check in the same way as concurrent firing above. Runs for different invoices proceed independently and never queue behind each other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules) | References (inbound) | Supplies the invoice's `due_date`, the anchor for the day-3/day-10 count |
| FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule) | References (inbound) | Supplies the freelancer's time zone used to count elapsed days |
| FEAT-11.SPEC-002 (Reminder Eligibility Rule) | Triggers (outbound) | Every candidate send is gated by this rule's re-check immediately before sending |
| FEAT-11.SPEC-004 (Overdue Reminder Email) | Triggers (outbound) | An eligible day-3 or day-10 candidate is handed off here to compose and send the email |
| FEAT-11.SPEC-003 (Invoice Reminder Panel) | Affects (outbound) | Every created Reminder Log entry appears in this panel's history |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Affects (outbound) | Every automatic send writes a trail entry (XBR-05) |
| FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | References (inbound) | The invoice's paid status, read at the eligibility re-check, is set by this spec |

## Analytics and Success Signals

- **reminder_auto_sent** (reminder_type: day_3 / day_10) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"
- **reminder_auto_send_skipped** (reminder_type: day_3 / day_10; reason: paid / paused_by_freelancer / paused_bank_transfer_pending) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"
- **reminder_auto_send_failed** (reminder_type: day_3 / day_10; retry_count) -- supports success-metrics.md: "Notification Delivery Reliability"

## Acceptance Criteria

**FEAT-11.SPEC-001-AC-01:** Given Nadia has an invoice for Owen that reaches exactly 3 elapsed days unpaid past its due date (freelancer's time zone), when the schedule evaluates that invoice, then a `day 3` Reminder Log entry is created, the eligibility check passes, and the Overdue Reminder Email is sent to Owen.

**FEAT-11.SPEC-001-AC-02:** Given the same invoice reaches exactly 10 elapsed days still unpaid, when the schedule evaluates it, then a `day 10` Reminder Log entry is created and sent, and no further automatic reminder is ever scheduled for this invoice.

**FEAT-11.SPEC-001-AC-03:** Given an invoice reaches day 3 but Owen paid it one minute before the schedule's eligibility re-check runs, when the re-check executes, then the send is skipped, no Reminder Log entry is created, and no email is sent.

**FEAT-11.SPEC-001-AC-04:** Given an invoice reaches day 3 while Nadia has paused reminders for it, when the schedule evaluates it, then the send is skipped per FEAT-11.SPEC-002 and no Reminder Log entry is created for day 3.

**FEAT-11.SPEC-001-AC-05:** Given an invoice reaches day 10 while it is Paused while bank transfer pending, when the schedule evaluates it, then the send is skipped, and the invoice never receives the day-10 reminder even if the pending transfer later fails and the invoice becomes unpaid again after day 10 has passed.

**FEAT-11.SPEC-001-AC-06:** Given an invoice's elapsed-day count is 5 (between the two thresholds), when the schedule evaluates it, then no reminder action is taken.

**FEAT-11.SPEC-001-AC-07:** Given the Overdue Reminder Email's hand-off fails on the first attempt for a day-3 candidate, when the automation retries per FEAT-14.SPEC-001's retry rule and the retry succeeds, then the Reminder Log entry's `sent_at` is set on the successful attempt and no delivery warning is shown to Nadia.

**FEAT-11.SPEC-001-AC-08:** Given the Overdue Reminder Email's hand-off fails on every retry attempt, when the final retry is exhausted, then the failure is logged rather than dropped silently and Nadia sees a delivery warning on the affected project.

**FEAT-11.SPEC-001-AC-09:** Given the schedule evaluation was unavailable for two days and an invoice passed day 3 unsent during that gap, when the schedule next runs, then the day-3 candidate is still detected (no existing `day 3` entry) and sent, provided the invoice is still eligible at that later moment.

**FEAT-11.SPEC-001-AC-10:** Given two schedule evaluations fire for the same invoice's day-3 threshold at effectively the same time, when both attempt to create the `day 3` entry, then only one Reminder Log entry is created and only one email is sent -- the second evaluation's dedup check finds the existing entry and takes no action.

**FEAT-11.SPEC-001-AC-11:** Given an invoice's day-3 threshold was skipped because Nadia had paused reminders, and she resumes before day 10, when the invoice reaches day 10 elapsed, then the eligibility check passes and the day-10 reminder sends normally -- the earlier skip does not block the later threshold.

**FEAT-11.SPEC-001-AC-12:** Given an invoice is Refunded before ever reaching day 3, when the schedule evaluates invoices, then this invoice is excluded from candidacy entirely, since its status is no longer one that can be overdue.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (day-3 threshold, day-10 threshold) | 2 |
| Outcome Paths | 5 (day-3 sent, day-10 sent, skipped-not-eligible, no-action, send failed) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Reminder Eligibility Rule

## Overview

**Name:** Reminder Eligibility Rule
**ID:** FEAT-11.SPEC-002
**Type:** Logic/Rule
**Purpose:** Defines whether a reminder -- automatic or manual -- may send right now: the paid check, the pause checks, the time-zone day-counting basis, and the one-manual-reminder-per-day limit, plus every role's authority over pausing, resuming, and manually sending.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders
**Governed Entity:** Reminder Log (including the per-invoice `pause_state` it carries and the derived Overdue status it feeds back to Invoice)

## Scope and Non-Goals

**In Scope:**
- The single "may this reminder send now" check: paid, freelancer-pause, and bank-transfer-pending-pause conditions
- Deriving elapsed overdue days from the invoice's `due_date` using the freelancer's time zone, and deriving the invoice's Overdue status from that count
- The one-manual-reminder-per-invoice-per-day limit
- Authorization for pausing, resuming, sending a manual reminder, and viewing reminder history/state, across every role in the Access Matrix
- Precedence between the two pause reasons (freelancer pause and bank-transfer-pending pause) when both could apply

**Non-Goals:**
- Deciding *when* day-3 and day-10 fall, and creating or sending the automatic reminder itself -- owned by FEAT-11.SPEC-001 (Reminder Schedule), which calls this rule immediately before each send.
- The screen behavior for pausing, resuming, or manually sending (button placement, feedback text, loading states) -- owned by FEAT-11.SPEC-003 (Invoice Reminder Panel), which enforces the rules defined here.
- Setting `pause_state` to Paused while bank transfer pending in the first place, and clearing it on resolution -- owned by FEAT-11.SPEC-005 (Bank-Transfer-Pending Reminder Pause); this spec defines only what the pause_state values *mean* for eligibility, not what sets them.
- A configurable reminder cadence or rate-limit value the freelancer can adjust -- excluded per scope-boundaries.md (SC-11); the one-per-day manual limit is a fixed product decision, not a setting.

## Governed Entity

**Entity:** Reminder Log
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| invoice | reference | The Invoice this log entry belongs to |
| reminder_type | enum (day 3, day 10, manual) | Which reminder point this entry records |
| scheduled_for | date/time (freelancer's time zone) | The computed point at which this reminder was due to fire |
| sent_at | date/time | When the reminder was actually sent; empty until send completes |
| pause_state | enum (Active, Paused by freelancer, Paused while bank transfer pending) | The invoice's current reminder-schedule state, carried on the Reminder Log per the dependency map; tracked once per invoice rather than once per entry, since it describes the schedule's live state, not a historical send |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-11.SPEC-001 | Reminder Schedule | Immediately before every automatic day-3/day-10 send |
| FEAT-11.SPEC-003 | Invoice Reminder Panel | On screen load (to show/hide/disable the Pause, Resume, and Send Reminder Now controls) and immediately before a manual send commits |
| FEAT-11.SPEC-005 | Bank-Transfer-Pending Reminder Pause | When applying or clearing the bank-transfer-pending pause, to resolve precedence against a freelancer pause |
| FEAT-31.SPEC-002 | Operator Support Session Console | On screen load, to render Dana's read-only mirror of the same panel with every write control disabled |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| invoice | Required; must reference an existing Invoice | Always | On create | -- (system-set at creation, never user-entered) | Yes |
| reminder_type | Required; must be exactly one of day 3, day 10, manual | Always | On create | -- (system-set at creation, never user-entered) | Yes |
| scheduled_for | No validation beyond data type -- always system-derived, never user input | Always | -- | -- | -- |
| sent_at | No validation beyond data type -- empty until the send completes, then system-set | Always | -- | -- | -- |
| pause_state | Required; must be exactly one of Active, Paused by freelancer, Paused while bank transfer pending | Always | On any transition | "Reminders are paused for this invoice." (surfaced by FEAT-11.SPEC-003 when a send attempt is blocked by a non-Active state) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Eligible-to-send | invoice.status, pause_state, existing Reminder Log entries for the invoice, reminder_type | A reminder (automatic or manual) may send only when all of: (a) the invoice's status is not Paid, Refunded, Partially refunded, or Corrected; (b) `pause_state` is Active; (c) if `reminder_type` is manual, no other manual entry for this invoice has `sent_at` within the current calendar day in the freelancer's time zone | Manual attempt while ineligible on (a): control not shown (nothing to remind about). Manual attempt while ineligible on (b): "Reminders are paused for this invoice. Resume reminders to send one now." Manual attempt while ineligible on (c): "You've already sent a reminder for this invoice today. You can send another tomorrow." |
| Overdue-day derivation | invoice.due_date, Freelancer Account.time_zone, invoice.status | Elapsed overdue days = whole calendar days between `due_date` and the current date, both evaluated in the freelancer's stored time zone; the invoice's Overdue status applies whenever elapsed days ≥ 1 and the invoice is not Paid, Refunded, Partially refunded, or Corrected | Not user-facing -- this is a system derivation with no error state |
| Pause-reason precedence | pause_state (current value), the pause reason being applied | A freelancer-initiated pause (FEAT-11.SPEC-003) always sets `pause_state` to Paused by freelancer, overwriting any existing Paused while bank transfer pending value. A bank-transfer-pending transition (FEAT-11.SPEC-005) sets `pause_state` to Paused while bank transfer pending only when the current value is Active, and clears it back to Active only when the current value is Paused while bank transfer pending -- it never touches a Paused by freelancer value in either direction | Not user-facing -- this governs which automated writer may change the field, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| View reminder history and current pause state | Nadia (Freelancer) | Always, for any of her own invoices | -- |
| View reminder history and current pause state | Dana (Support Operator) | Only inside a logged, time-limited support session opened on the freelancer's account (FEAT-31.SPEC-002) | Outside an open support session, the panel is unreachable to Dana at all |
| View reminder history and current pause state | Owen (Client Primary Contact) | Never | The Invoice Reminder Panel is never shown to Owen; he receives only the Overdue Reminder Email (FEAT-11.SPEC-004) |
| View reminder history and current pause state | Priya (Client Reviewer Contact) | Never | The panel is never shown to Priya, and she is never addressed by any reminder email (consistent with her having no Invoicing & Payments access) |
| Pause reminders for an invoice | Nadia (Freelancer) | Always, for any of her own invoices, regardless of the invoice's current pause reason | -- |
| Pause reminders for an invoice | Dana (Support Operator) | Never | The Pause control is rendered but disabled in Dana's read-only mirror, with the experience "Pause is unavailable in a read-only support session." |
| Pause reminders for an invoice | Owen, Priya | Never | Control not shown -- neither contact reaches this panel |
| Resume reminders for an invoice | Nadia (Freelancer) | Only when `pause_state` is currently Paused by freelancer (resuming a bank-transfer-pending pause is not a freelancer action -- see FEAT-11.SPEC-005) | If `pause_state` is currently Paused while bank transfer pending, the Resume control shows "Paused while a bank transfer is pending -- this resumes automatically." instead of an actionable button |
| Resume reminders for an invoice | Dana (Support Operator) | Never | Control disabled with "Resume is unavailable in a read-only support session." |
| Resume reminders for an invoice | Owen, Priya | Never | Control not shown |
| Send a manual reminder | Nadia (Freelancer) | Only when the Eligible-to-send cross-field rule passes (invoice not paid/refunded/corrected, `pause_state` is Active, no manual send already sent today for this invoice) | See the Eligible-to-send rule's per-condition denied messages above |
| Send a manual reminder | Dana (Support Operator) | Never | Control disabled with "Sending is unavailable in a read-only support session." |
| Send a manual reminder | Owen, Priya | Never | Control not shown |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| pause_state | Active | When an invoice first becomes eligible to carry a reminder schedule (i.e., once it is sent, per FEAT-09) | Yes -- Nadia may pause at any time (Authorization Rules); the automatic bank-transfer-pending trigger (FEAT-11.SPEC-005) may also change it, subject to the Pause-reason precedence rule |
| scheduled_for | `due_date` + 3 calendar days (for `reminder_type` = day 3) or `due_date` + 10 calendar days (for day 10), computed in the freelancer's time zone; not applicable to manual entries, which record `scheduled_for` as the moment Nadia triggered the send | On creation of each Reminder Log entry (FEAT-11.SPEC-001 or FEAT-11.SPEC-003) | No |
| Invoice.status = Overdue | Derived: true whenever elapsed overdue days ≥ 1 and the invoice is not Paid, Refunded, Partially refunded, or Corrected | Continuously, re-evaluated at every eligibility check and at invoice display | No -- this status is never set directly by any role; it is always computed |

## Business Rules

- XBR-15: reminders stop the instant the invoice is paid, pause while a bank transfer is pending or when the freelancer pauses that invoice, and manual reminders are limited to one per invoice per day.
- XBR-20: an invoice is paid once and in full only, and processor-confirmed payment status is authoritative -- the paid check in the Eligible-to-send rule always reflects the most recently confirmed status (FEAT-10.SPEC-004), never a stale read.
- The dependency map's Contention note for Reminder Log applies: the schedule re-checks pause, pending, and paid state immediately before each send; a pause saved first wins.
- The one-manual-reminder-per-day limit is measured against the freelancer's own local calendar day (FEAT-15.SPEC-006), not a fixed 24-hour rolling window -- a manual reminder sent at 11:58pm and another attempted at 12:02am the same freelancer-local night are on different calendar days and both may proceed.
- A second concurrent manual send attempt against the same invoice is refused with refresh (dependency map, Reminder Log Contention note): the second request's Eligible-to-send check reads the entry the first request just created and denies it under the one-per-day limit.

## Edge Cases

- **Invoice reaches Paid and a manual send is attempted in the same moment** -- The Eligible-to-send check reads the invoice's current (already-Paid) status and denies the send; the control is also hidden on the next panel refresh once Paid is visible.
- **Both pause reasons would apply at once (Nadia pauses while a bank transfer is already pending)** -- The Pause-reason precedence rule resolves this: Nadia's action always wins and sets Paused by freelancer, overwriting Paused while bank transfer pending. If the bank transfer later resolves (FEAT-11.SPEC-005), the pending-pause clearing logic finds the current value is Paused by freelancer, not Paused while bank transfer pending, and does not touch it -- the invoice remains paused until Nadia resumes it herself.
- **Manual send exactly at the boundary of the freelancer's local calendar day** -- A send at 23:59:59 and a second attempt at 00:00:01 the same freelancer-local night are on different calendar days per the one-per-day rule's basis; the second attempt is allowed.
- **Elapsed-day count crosses a daylight-saving transition in the freelancer's time zone** -- The calendar-day count is unaffected; a transition day still counts as exactly one elapsed day, consistent with FEAT-15.SPEC-006's local-date-boundary rule.
- **Dana opens a support session on an invoice that Nadia has paused** -- Dana sees the Paused by freelancer state exactly as Nadia would, but every action control (Pause, Resume, Send Reminder Now) remains disabled per the Authorization Rules -- read-only is unconditional and does not vary with the invoice's pause state.
- **Nadia's session pauses an invoice while the Reminder Schedule (FEAT-11.SPEC-001) has already begun its eligibility check for the same invoice's day-3 send** -- Whichever write commits first wins: if the pause commits before the schedule's eligibility read, the send is denied (skipped); if the send has already been recorded as sent before the pause commits, the completed send stands and the pause applies only to future thresholds.

## Acceptance Criteria

**FEAT-11.SPEC-002-AC-01:** Given Nadia's invoice to Owen is Overdue with `pause_state` Active and no reminder sent today, when the Eligible-to-send rule is checked for a manual send, then it returns eligible.

**FEAT-11.SPEC-002-AC-02:** Given the same invoice has just been marked Paid, when the Eligible-to-send rule is checked (automatic or manual), then it returns not eligible and the Send Reminder Now control is not shown on FEAT-11.SPEC-003.

**FEAT-11.SPEC-002-AC-03:** Given the invoice's `pause_state` is Paused by freelancer, when a manual send is attempted, then it is denied with "Reminders are paused for this invoice. Resume reminders to send one now."

**FEAT-11.SPEC-002-AC-04:** Given Nadia already sent a manual reminder for this invoice earlier today (freelancer's local day), when she attempts to send another today, then it is denied with "You've already sent a reminder for this invoice today. You can send another tomorrow."

**FEAT-11.SPEC-002-AC-05:** Given Nadia sent a manual reminder at 23:59 freelancer-local time, when she attempts another at 00:02 the following freelancer-local day, then the send is allowed, since the two attempts fall on different calendar days.

**FEAT-11.SPEC-002-AC-06:** Given an invoice's `due_date` was yesterday in the freelancer's time zone, when the Overdue-day derivation runs, then the invoice's status is Overdue with elapsed days = 1.

**FEAT-11.SPEC-002-AC-07:** Given an invoice is Refunded, when the Overdue-day derivation runs, then the invoice's status is never set to Overdue, regardless of how many days have elapsed since its due date.

**FEAT-11.SPEC-002-AC-08:** Given an invoice's `pause_state` is currently Active, when the Bank-Transfer-Pending Reminder Pause (FEAT-11.SPEC-005) applies its pause, then `pause_state` becomes Paused while bank transfer pending.

**FEAT-11.SPEC-002-AC-09:** Given an invoice's `pause_state` is currently Paused by freelancer, when a bank transfer for that invoice enters pending status, then `pause_state` remains Paused by freelancer -- the automatic pause never overwrites Nadia's own pause.

**FEAT-11.SPEC-002-AC-10:** Given an invoice's `pause_state` is Paused while bank transfer pending, when Nadia taps Pause on FEAT-11.SPEC-003, then `pause_state` becomes Paused by freelancer, overwriting the automatic pause reason.

**FEAT-11.SPEC-002-AC-11:** Given an invoice's `pause_state` is Paused while bank transfer pending, when the pending transfer resolves to Paid or reverts, then `pause_state` returns to Active (subject to the invoice's paid check for the Paid outcome).

**FEAT-11.SPEC-002-AC-12:** Given Nadia (Freelancer) opens the Invoice Reminder Panel for her own invoice, when the panel loads, then she can view the full reminder history and pause state, and the Pause/Resume and Send Reminder Now controls are active per her invoice's current state.

**FEAT-11.SPEC-002-AC-13:** Given Dana (Support Operator) has an open support session on the freelancer's account, when she opens the same panel, then she can view the same history and state read-only, and every action control appears disabled with "unavailable in a read-only support session."

**FEAT-11.SPEC-002-AC-14:** Given Owen (Client Primary Contact) is signed in to his own portal, when he looks for any path to the Invoice Reminder Panel, then none exists -- the panel is never shown to him.

**FEAT-11.SPEC-002-AC-15:** Given Priya (Client Reviewer Contact) is signed in to her own portal, when she looks for any path to the panel or for a reminder email addressed to her, then neither exists.

**FEAT-11.SPEC-002-AC-16:** Given Nadia has paused reminders and two of her own browser sessions both attempt to send a manual reminder for the same invoice at effectively the same time, when both requests reach the Eligible-to-send check, then at most one succeeds and the second is refused with refresh, per the one-per-day limit reading the first request's just-created entry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Invoice Reminder Panel

## Overview

**Name:** Invoice Reminder Panel
**ID:** FEAT-11.SPEC-003
**Type:** Screen
**Purpose:** Shows an invoice's reminder history and lets Nadia pause or resume the automatic reminder schedule and send a manual reminder, all scoped to one invoice.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders

## Scope and Non-Goals

**In Scope:**
- Displaying the invoice's Reminder Log history (day 3, day 10, and manual entries, in send order) with each entry's type and timestamp in Nadia's own time zone
- Displaying the invoice's current pause state
- Pausing and resuming the reminder schedule for this one invoice
- Sending a manual reminder, gated by FEAT-11.SPEC-002's eligibility rule
- Dana's read-only mirror of this same panel inside a logged support session

**Non-Goals:**
- Determining the day-3/day-10 schedule dates or performing the automatic sends themselves -- owned by FEAT-11.SPEC-001 (Reminder Schedule).
- The paid/pause/rate-limit eligibility logic itself -- owned by FEAT-11.SPEC-002 (Reminder Eligibility Rule); this panel enforces what that spec decides, it does not decide it.
- The content of the reminder email Owen receives -- owned by FEAT-11.SPEC-004 (Overdue Reminder Email).
- A standalone landing screen or list of reminders across invoices -- excluded per this feature's Shared Context: this feature has no cross-invoice reminder list; the invoice detail (FEAT-09.SPEC-002) is the only surface, consistent with the product's per-invoice, no-configurable-automation design (scope-boundaries.md, SC-11).

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Nadia opens an invoice's detail view; this panel is embedded as a section within it | The invoice reference and its current status |
| FEAT-12.SPEC-001 (Dashboard Overview) | Nadia clicks an invoice flagged Overdue on her dashboard | Navigates to FEAT-09.SPEC-002 for that invoice, with this panel visible in place |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Dana opens a logged support session and navigates to the same invoice's detail on the freelancer's mirrored account | The invoice reference; every control on this panel renders disabled |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full panel: reminder history, current pause state | Pause, resume, and send a manual reminder, each gated by FEAT-11.SPEC-002 | -- |
| Owen (Client Primary Contact) | No | No | The panel does not exist anywhere in Owen's portal; he receives only the Overdue Reminder Email (FEAT-11.SPEC-004) and its pay link |
| Priya (Client Reviewer Contact) | No | No | The panel does not exist anywhere in Priya's portal, consistent with her having no Invoicing & Payments access at all |
| Dana (Support Operator) | Full panel, read-only, only inside a logged, time-limited support session (FEAT-31.SPEC-002) | None -- every control renders disabled with "unavailable in a read-only support session" | Outside an open support session, Dana has no path to this panel at all |
| Unauthenticated | No | No | Redirected to the freelancer sign-in screen; after signing in, the user lands on FEAT-12.SPEC-001 (Dashboard Overview), not this panel directly |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- if a pause/resume or manual-send action was mid-flight, it is discarded and must be retried after re-authentication |

## Layout and Content

**Section placement:** This panel is a "Reminders" section within FEAT-09.SPEC-002 (Invoice Detail), positioned below the invoice's payment status area.

**Header:** Section title "Reminders" with the current pause-state indicator to its right (a short label: "Active", "Paused by you", or "Paused -- bank transfer pending").

**Body:**
- A "Send Reminder Now" button, top-right of the section, active whenever FEAT-11.SPEC-002's eligibility rule allows a manual send for this invoice.
- A "Pause reminders" / "Resume reminders" toggle button, next to the pause-state indicator, whose label and behavior depend on the current `pause_state`: "Pause reminders" while Active, "Resume reminders" while Paused by freelancer, and "Pause reminders" (remaining tap-able) while Paused while bank transfer pending.
- Below the controls, a reminder history list, one row per Reminder Log entry, in send order (oldest first): reminder type (Day 3, Day 10, or Manual), and the date/time it was sent, in Nadia's own time zone. Dana's mirrored view shows the same rows.
- If `pause_state` is Paused while bank transfer pending, static text appears next to the toggle: "Paused while a bank transfer is pending -- this resumes automatically." The toggle itself remains present and reads "Pause reminders" -- tapping it invokes the same Pause action as the Active state, per FEAT-11.SPEC-002-AC-10. There is no Resume control in this state; resuming happens only automatically when the bank transfer resolves (FEAT-11.SPEC-005).

All fields use one consistent input treatment platform-wide, per the design layer.

### Responsive Behavior

- **Compact breakpoint:** The pause-state indicator and toggle stack below the "Reminders" header; "Send Reminder Now" remains a full-width button below them; history rows stack as single-column cards.
- **Medium size class and above:** Header, pause-state indicator, toggle, and "Send Reminder Now" sit on one row; history rows render as a compact table (type, timestamp).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Pause reminders toggle | Tap (while `pause_state` is Active) | Invokes FEAT-11.SPEC-002's authorization check, then sets `pause_state` to Paused by freelancer | Indicator switches to "Paused by you"; toggle label switches to "Resume reminders" | Toast: "Reminders paused for this invoice." |
| Pause reminders toggle | Tap (while `pause_state` is Paused while bank transfer pending) | Invokes FEAT-11.SPEC-002's authorization check, then sets `pause_state` to Paused by freelancer, overwriting the automatic pause reason (FEAT-11.SPEC-002-AC-10) | Indicator switches to "Paused by you"; the "Paused while a bank transfer is pending" static text is removed; toggle label switches to "Resume reminders" | Toast: "Reminders paused for this invoice." |
| Resume reminders toggle | Tap (while `pause_state` is Paused by freelancer) | Invokes FEAT-11.SPEC-002's authorization check, then sets `pause_state` to Active | Indicator switches to "Active"; toggle label switches to "Pause reminders" | Toast: "Reminders resumed for this invoice." |
| Send Reminder Now button | Tap | Invokes FEAT-11.SPEC-002's Eligible-to-send check; if eligible, triggers FEAT-11.SPEC-004 (Overdue Reminder Email) with `reminder_type` = manual, then creates the Reminder Log entry | Button shows a brief loading state during the check and send; a new "Manual" row appears at the bottom of the history on success | Success: toast "Reminder sent to {Primary Contact name}." and the new history row appears. Denied: exact message from FEAT-11.SPEC-002's Authorization Rules (e.g. "You've already sent a reminder for this invoice today. You can send another tomorrow.") shown inline near the button; no history row is added. |
| Send Reminder Now button (while loading) | Tap | No action -- debounced | Button remains in loading state | Button stays disabled until the in-flight attempt resolves |
| Reminder history row | -- | Display-only -- no interaction | None | -- |

### Accessibility Notes

- **Focus order:** Pause-state indicator -> Pause/Resume toggle -> Send Reminder Now button -> reminder history rows (in send order).
- **Dynamic announcements:** The pause-state indicator's change is announced to assistive technology when the toggle completes. The "Reminder sent" toast and any denied-send message are announced on appearance.
- **Keyboard alternatives:** The toggle and Send Reminder Now button are both standard activatable controls reachable and operable entirely by keyboard; there are no pointer-only gestures on this panel.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| No reminders yet | History area shows "No reminders sent yet -- automatic reminders begin 3 days after the due date if this invoice stays unpaid." Pause/Resume and Send Reminder Now controls remain visible per the current eligibility state | Invoice has no Reminder Log entries (not yet overdue, or overdue but not yet at day 3) | The first Reminder Log entry (automatic or manual) is created |
| Populated | History list shows one or more entries; pause-state indicator reflects the current value | At least one Reminder Log entry exists | Never exits -- remains populated once the first entry exists |
| Loading | Panel shows a loading placeholder in place of the history list and controls | Panel first mounts, before the invoice's Reminder Log data has loaded | Data load completes (success or error) |
| Error | Banner: "Couldn't load reminder history. Try again." with a Retry control; controls (Pause/Resume/Send) are disabled until retried | The reminder-history data load fails | Nadia taps Retry and the load succeeds |
| Action in progress | The specific control (toggle or Send Reminder Now) that was tapped shows a loading state; other controls remain interactive | Nadia taps Pause, Resume, or Send Reminder Now | The action completes (success or denied) |
| Offline/Degraded | Banner "You're offline -- reminder history is shown from your last successful load. Pause, resume, and manual sends are unavailable until you reconnect." History remains visible read-only; all action controls are disabled | Connectivity lost while the panel is open | Connectivity restored -- controls re-enable and the panel re-fetches the latest state |

## Validation Rules

Validation and authorization for pausing, resuming, and manual sending are governed entirely by FEAT-11.SPEC-002 (Reminder Eligibility Rule). See that spec for every allowed/denied condition and its exact denied message. This panel applies those checks at the moment each control is activated, never before.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| This panel has no navigation controls of its own -- it is a section within FEAT-09.SPEC-002 | FEAT-09.SPEC-002 (Invoice Detail) | -- (same screen; this is a section, not a distinct navigable screen) |

## Data Model

**Creates:** Reminder Log entry -- `invoice`, `reminder_type` = manual, `scheduled_for` = the moment of the tap, `sent_at` set on successful send.
**Reads:** Invoice -- `status`, `due_date` (for context and to gate eligibility, from FEAT-09/FEAT-10). Reminder Log -- every entry for this invoice, in send order. Client Contact -- the invoice's Primary Contact name, shown in the "Reminder sent to {name}" feedback.
**Updates:** Reminder Log -- `pause_state`, via the Pause/Resume toggle.
**Deletes:** None -- this panel never removes a Reminder Log entry.

## Business Rules

- FEAT-11.SPEC-002 owns every eligibility, authorization, and rate-limit rule enforced on this panel; this panel never re-implements those checks.
- XBR-15: manual reminders are limited to one per invoice per day, enforced by FEAT-11.SPEC-002 and surfaced here as a denied message rather than a hidden control, so Nadia understands why the button is unavailable.
- Pausing or resuming affects only the one invoice this panel is scoped to -- there is no bulk pause/resume across invoices, consistent with the product's fixed, per-invoice behavior (SC-11).

## Edge Cases

- **Nadia taps Send Reminder Now twice in rapid succession** -- The second tap is ignored while the first is in flight (button in loading state, debounced).
- **The automatic Reminder Schedule (FEAT-11.SPEC-001) sends a day-3 reminder while this panel is open** -- The panel is a snapshot at load time; the new history row does not appear live. Nadia sees it on her next visit or manual refresh. This is acceptable because reminder sending has no time-sensitive interactive component the freelancer must react to in the moment.
- **Nadia taps Pause at the exact moment the automatic schedule's eligibility re-check runs for a threshold that is currently due** -- Per the dependency map's Contention note (a pause saved first wins): if Nadia's pause commits first, the automatic send is skipped; if the automatic send has already completed, the pause takes effect only for the next threshold.
- **Two of Nadia's own sessions both tap Send Reminder Now for the same invoice at effectively the same time** -- Resolution: reject-with-refresh, per the dependency map's Contention note for Reminder Log. The first request to commit succeeds; the second's eligibility check reads the just-created entry and is denied under the one-per-day limit, with the panel refreshing to show the new entry.
- **Nadia navigates away mid-send (before the manual-send result returns) and comes back** -- On return, the panel re-fetches the current state; if the send had actually completed on the server before she navigated away, the new history row is present. No duplicate send is triggered by returning to the panel.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (inbound) | This panel is embedded as a section within that screen |
| FEAT-12.SPEC-001 (Dashboard Overview) | Navigation (inbound) | Nadia arrives here after clicking an Overdue invoice on her dashboard |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Navigation (inbound) | Dana's mirrored, read-only view of this same panel |
| FEAT-11.SPEC-001 (Reminder Schedule) | References (inbound) | Automatic sends populate the history this panel displays |
| FEAT-11.SPEC-002 (Reminder Eligibility Rule) | References (inbound) | Every action on this panel is authorized and validated by that spec |
| FEAT-11.SPEC-004 (Overdue Reminder Email) | Triggers (outbound) | The Send Reminder Now action triggers this notification |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Affects (outbound) | Pause, resume, and manual-send actions each write a trail entry (XBR-05) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| reminder_paused | invoice reference, elapsed overdue days at time of pause | Nadia successfully pauses reminders for an invoice | supports success-metrics.md: "Reminder-Driven Payment Recovery" |
| reminder_resumed | invoice reference | Nadia successfully resumes reminders for an invoice | supports success-metrics.md: "Reminder-Driven Payment Recovery" |
| manual_reminder_sent | invoice reference, elapsed overdue days at time of send | Nadia's manual send completes successfully | supports success-metrics.md: "Reminder-Driven Payment Recovery" |
| manual_reminder_send_denied | invoice reference, denial reason (paid / paused / rate_limited) | A manual send attempt is denied | supports success-metrics.md: "Reminder-Driven Payment Recovery" |

## Acceptance Criteria

**FEAT-11.SPEC-003-AC-01:** Given Nadia opens the Invoice Detail for an Overdue invoice that has already had its day-3 reminder sent, when the panel loads, then she sees one "Day 3" history row with its send timestamp in her own time zone.

**FEAT-11.SPEC-003-AC-02:** Given Nadia opens the panel for an invoice not yet at day 3, when the panel loads, then she sees "No reminders sent yet -- automatic reminders begin 3 days after the due date if this invoice stays unpaid."

**FEAT-11.SPEC-003-AC-03:** Given Nadia taps "Pause reminders" on an Active invoice, when the action completes, then the indicator switches to "Paused by you", the toggle becomes "Resume reminders", and she sees the toast "Reminders paused for this invoice."

**FEAT-11.SPEC-003-AC-04:** Given the invoice's `pause_state` is Paused while bank transfer pending, when Nadia views the panel, then she sees the static text "Paused while a bank transfer is pending -- this resumes automatically." next to a "Pause reminders" toggle that remains tap-able (there is no Resume button in this state).

**FEAT-11.SPEC-003-AC-05:** Given the invoice is eligible for a manual reminder, when Nadia taps "Send Reminder Now", then the reminder is sent, a new "Manual" row appears in the history, and she sees "Reminder sent to Owen." (or the Primary Contact's actual name).

**FEAT-11.SPEC-003-AC-06:** Given Nadia already sent a manual reminder for this invoice earlier today, when she taps "Send Reminder Now" again, then she sees the inline message "You've already sent a reminder for this invoice today. You can send another tomorrow." and no history row is added.

**FEAT-11.SPEC-003-AC-07:** Given Owen (Client Primary Contact) is viewing his own portal, when he looks for any way to reach this panel, then no such path exists anywhere in his portal.

**FEAT-11.SPEC-003-AC-08:** Given Dana (Support Operator) has an open support session on the freelancer's account, when she opens this panel, then she sees the full history and pause state, but every control (Pause, Resume, Send Reminder Now) is disabled with "unavailable in a read-only support session."

**FEAT-11.SPEC-003-AC-09:** Given the reminder-history data fails to load, when the panel attempts its initial load, then Nadia sees "Couldn't load reminder history. Try again." with a Retry control, and all action controls are disabled until the retry succeeds.

**FEAT-11.SPEC-003-AC-10:** Given Nadia loses connectivity while the panel is open, when she attempts to tap Pause, then the offline banner is shown and the action is not attempted until connectivity returns.

**FEAT-11.SPEC-003-AC-11:** Given Nadia taps "Send Reminder Now" twice in rapid succession, when the first tap is still processing, then the second tap has no effect and the button remains in its loading state.

**FEAT-11.SPEC-003-AC-12:** Given Nadia's session expires while a Pause action is mid-flight, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and the pause action is discarded, requiring her to retry after re-authenticating.

**FEAT-11.SPEC-003-AC-13:** Given two of Nadia's own sessions both tap "Send Reminder Now" for the same invoice at effectively the same time, when both requests are processed, then only one manual send succeeds and the other is denied with the one-per-day message, with its panel refreshing to show the new entry.

**FEAT-11.SPEC-003-AC-14:** Given the invoice becomes Paid while Nadia is viewing the panel, when she next taps "Send Reminder Now" (without having refreshed), then the send attempt is denied per FEAT-11.SPEC-002 rather than silently succeeding, since eligibility is re-checked at the moment of the tap.

**FEAT-11.SPEC-003-AC-15:** Given Priya (Client Reviewer Contact) is viewing her own portal, when she looks for any way to reach this panel or receive a reminder-related communication, then neither exists anywhere in her portal.

**FEAT-11.SPEC-003-AC-16:** Given the invoice's `pause_state` is Paused while bank transfer pending, when Nadia taps the "Pause reminders" toggle, then `pause_state` becomes Paused by freelancer (overwriting the automatic pause reason, per FEAT-11.SPEC-002-AC-10), the static "Paused while a bank transfer is pending" text is removed, the indicator switches to "Paused by you", the toggle switches to "Resume reminders", and she sees the toast "Reminders paused for this invoice."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (no reminders yet, populated, loading, error, action in progress, offline/degraded) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Notification Spec: Overdue Reminder Email

## Overview

**Name:** Overdue Reminder Email
**ID:** FEAT-11.SPEC-004
**Type:** Notification
**Purpose:** Sends the client's Primary Contact a polite email about an overdue invoice, on the day-3 automatic reminder, the day-10 automatic reminder, or Nadia's manual reminder -- with a direct way to pay.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders

## Scope and Non-Goals

**In Scope:**
- The email sent to the invoice's Primary Contact for a day-3 automatic, day-10 automatic, or manual reminder
- The three content variants (day 3, day 10, manual) and their shared pay-link CTA
- Delivery, retry, and expiry behavior for this email

**Non-Goals:**
- Deciding when day-3 and day-10 fall, or whether a given send is currently eligible -- owned by FEAT-11.SPEC-001 (Reminder Schedule) and FEAT-11.SPEC-002 (Reminder Eligibility Rule); this spec begins once one of those has already decided to send.
- Any channel beyond email -- excluded per assumptions-constraints.md (ASMP-29) and BRIEF.md's Ecosystem & Integrations: "clients will not install an app," making email the sole channel reaching client contacts; no SMS or push capability exists in the product definition.
- Addressing Priya (Client Reviewer Contact) or any contact other than the invoice's Primary Contact -- grounded in the Access Matrix: Priya's Invoicing & Payments access is None, and reminders are addressed only to the Primary Contact per this feature's Access field.
- The pay flow itself once Owen clicks the link -- owned by FEAT-10.SPEC-001 (Pay Invoice Screen).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always -- every day-3, day-10, or manual reminder is delivered by email | Email is the sole channel reaching client contacts (ASMP-29, BRIEF.md's Ecosystem & Integrations); Owen is not inside the product between sessions triggered by a specific link, so an interruption must reach him where he already is |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Day-3 automatic reminder eligible and ready to send | FEAT-11.SPEC-001 (Reminder Schedule) | Fires when the schedule's eligibility re-check for the day-3 threshold passes | Invoice reference, invoice number, amount, currency, due date, freelancer business name, Primary Contact reference, pay-link availability |
| Day-10 automatic reminder eligible and ready to send | FEAT-11.SPEC-001 (Reminder Schedule) | Fires when the schedule's eligibility re-check for the day-10 threshold passes | Same as above |
| Manual reminder eligible and ready to send | FEAT-11.SPEC-003 (Invoice Reminder Panel) | Fires when Nadia's "Send Reminder Now" action passes FEAT-11.SPEC-002's Eligible-to-send check | Same as above |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) -- the invoice's Primary Contact, per the Access Matrix's Invoicing & Payments row ("Own-only: view, pay, download copies") and XBR-08 (only Primary contacts see and pay invoices). Priya (Client Reviewer Contact) is never a recipient -- her Invoicing & Payments access is None. Dana (Support Operator) never receives this email; she may view its delivery status read-only inside a logged support session, mirroring the reminder history on FEAT-11.SPEC-003.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- not configurable | -- | Always sends | -- |

This email is a transactional record email core to the invoice's payment lifecycle, not an optional notification; per XBR-30, transactional emails core to the record always send and cannot be disabled by either Nadia or Owen.

**Quiet Hours:** N/A -- the product definition establishes no quiet-hours capability anywhere (no Stage 2 document defines one); every reminder sends the moment it is triggered.

## Content Definition

**Email (day 3):**
- **Subject:** Reminder: Invoice {invoice_number} from {freelancer_business_name} is now overdue
- **Body:**
  Hi {contact_first_name},

  This is a friendly reminder that invoice {invoice_number} for {amount_due} was due on {due_date} and hasn't been paid yet.

  You can take care of it in a couple of minutes using the link below.
- **CTA (button):** Pay Invoice {invoice_number} -- deep-links to FEAT-10.SPEC-001 (Pay Invoice Screen) for this invoice

**Email (day 10):**
- **Subject:** Second reminder: Invoice {invoice_number} from {freelancer_business_name} is still overdue
- **Body:**
  Hi {contact_first_name},

  Invoice {invoice_number} for {amount_due} (due {due_date}) is still showing as unpaid, {days_overdue} days after the due date.

  If there's anything holding this up, please reach out to {freelancer_business_name} directly -- otherwise, you can pay it now using the link below.
- **CTA (button):** Pay Invoice {invoice_number} -- deep-links to FEAT-10.SPEC-001 (Pay Invoice Screen) for this invoice

**Email (manual):**
- **Subject:** Reminder: Invoice {invoice_number} from {freelancer_business_name} is due
- **Body:**
  Hi {contact_first_name},

  {freelancer_business_name} wanted to remind you that invoice {invoice_number} for {amount_due} (due {due_date}) hasn't been paid yet.

  You can take care of it using the link below.
- **CTA (button):** Pay Invoice {invoice_number} -- deep-links to FEAT-10.SPEC-001 (Pay Invoice Screen) for this invoice

**No-pay-link variant (all three, applied when pay-link availability is unavailable per FEAT-09.SPEC-009):** The CTA button is replaced with the plain-text instructions for paying the freelancer directly, exactly as FEAT-09.SPEC-009 defines them for the original invoice email; the subject and body above are otherwise unchanged.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {contact_first_name} | Client Contact -- name (first token) | Owen | Never empty -- name is required at contact creation (FEAT-18) |
| {freelancer_business_name} | Freelancer Account -- business_name | Studio Nadia | Never empty -- business name is required before the first invoice is sent |
| {invoice_number} | Invoice -- invoice_number | INV-0042 | Never empty -- required and system-assigned at generation |
| {amount_due} | Invoice -- total, formatted with its currency | $1,200.00 | Never empty -- required at invoice generation |
| {due_date} | Invoice -- due_date, rendered in the recipient's own time zone per FEAT-15.SPEC-006 | March 3, 2026 | Never empty -- required before the invoice sends |
| {days_overdue} | Derived -- elapsed calendar days since due_date, computed in the freelancer's time zone (FEAT-11.SPEC-002) | 10 | Never empty -- the day-10 variant only renders once this value is exactly 10 |

## Delivery Rules

**Batching:** None -- each reminder is delivered as its own, single email tied to one invoice and one threshold. Reminders for two different overdue invoices are never combined into one email, since each invoice's overdue status and pay link are independent and combining them would blur which invoice needs attention.
**Deduplication:** At most one email per Reminder Log entry. A Reminder Schedule re-run (FEAT-11.SPEC-001) never re-sends the email tied to an already-`sent_at` entry; only a newly created entry (a new threshold, or a new manual send) produces a new email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the Reminder Log entry itself still shows as sent from Nadia's perspective in FEAT-11.SPEC-003's history, since the send was attempted and logged -- the delivery warning is the surviving signal of the failure, not a change to the history entry.
**Expiry:** N/A -- an overdue reminder has no meaningful expiry window distinct from the retry window itself: once the retry window is exhausted, the failure is reported per the rule above rather than the email being discarded as stale. A reminder is never held past its trigger moment awaiting a better time to send (no quiet hours, no batching window to wait out).

## Edge Cases

- **Invoice is paid moments after the eligibility check passes but before the email is actually delivered** -- The queued send is not cancelled once handed to the delivery capability; a reminder that arrives moments after payment is a rare, narrow timing artifact accepted as a product decision rather than treated as a defect, since eligibility was genuinely true at the moment the send was authorized.
- **The invoice's Primary Contact is removed (erasure request, FEAT-18) between trigger and delivery** -- The send is cancelled; there is no longer a recipient entitled to receive it. This is logged as a delivery-skipped-no-recipient outcome and surfaced to Nadia as a delivery warning on the affected project (XBR-30), since it is a break in the reminder chain she should know about.
- **No preference to collide with quiet hours or a preference change mid-flight** -- Because this email carries no preference control and the product defines no quiet hours (per Audience and Preferences above), the usual preference/quiet-hours collision edge cases do not apply to this notification; this is a resolved decision, not an omission.
- **Pay-link availability changes between trigger and delivery (Nadia's payment account moves from Connected to Needs attention)** -- The email renders with whichever pay-link state (linked CTA or plain-text instructions) is current at the moment of composition, immediately before send, consistent with FEAT-09.SPEC-009's authority over pay-link wording.
- **A day-3 and a day-10 reminder for two different invoices both fire for the same recipient on the same day** -- Each is delivered as its own separate email, per the no-batching rule; Owen receives two distinct emails, each naming its own invoice number.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-11.SPEC-001 (Reminder Schedule) | Triggered by (inbound) | Fires this notification for every eligible day-3 or day-10 send |
| FEAT-11.SPEC-003 (Invoice Reminder Panel) | Triggered by (inbound) | Fires this notification for every eligible manual send |
| FEAT-11.SPEC-002 (Reminder Eligibility Rule) | References (inbound) | Governs whether the triggering send was allowed to happen at all |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Used for the actual send and for delivery/bounce status reporting |
| FEAT-10.SPEC-001 (Pay Invoice Screen) | Navigation (outbound) | Every CTA deep-links here for the specific invoice |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (inbound) | Governs the CTA's linked-vs-plain-text rendering |
| FEAT-31.SPEC-002 (Operator Support Session Console) | References (outbound) | Dana views this email's delivery status read-only inside a logged support session |

## Analytics and Success Signals

- **reminder_email_delivered** (reminder_type: day_3 / day_10 / manual) -- supports success-metrics.md: "Notification Delivery Reliability"
- **reminder_email_delivery_failed** (reminder_type: day_3 / day_10 / manual; retry_count) -- supports success-metrics.md: "Notification Delivery Reliability"
- **reminder_pay_link_clicked** (reminder_type: day_3 / day_10 / manual) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"

## Acceptance Criteria

**FEAT-11.SPEC-004-AC-01:** Given Owen's invoice reaches day 3 overdue and eligibility passes, when FEAT-11.SPEC-001 fires this notification, then Owen receives an email with subject "Reminder: Invoice {invoice_number} from {freelancer_business_name} is now overdue" and a "Pay Invoice {invoice_number}" button.

**FEAT-11.SPEC-004-AC-02:** Given Owen's invoice reaches day 10 overdue and eligibility passes, when the notification fires, then Owen receives the day-10 email with subject "Second reminder: Invoice {invoice_number} from {freelancer_business_name} is still overdue" naming {days_overdue} as 10.

**FEAT-11.SPEC-004-AC-03:** Given Nadia sends a manual reminder and eligibility passes, when the notification fires, then Owen receives the manual variant with subject "Reminder: Invoice {invoice_number} from {freelancer_business_name} is due".

**FEAT-11.SPEC-004-AC-04:** Given Owen taps "Pay Invoice {invoice_number}" in any variant, when the link is followed, then he lands on FEAT-10.SPEC-001 for that specific invoice.

**FEAT-11.SPEC-004-AC-05:** Given Nadia has no connected, ready payment account for this invoice, when any reminder variant is sent, then the CTA is replaced with the plain-text pay-the-freelancer-directly instructions from FEAT-09.SPEC-009.

**FEAT-11.SPEC-004-AC-06:** Given the email fails to deliver on the first attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project for Nadia.

**FEAT-11.SPEC-004-AC-07:** Given the invoice's Primary Contact was removed via an erasure request between trigger and delivery, when the send is attempted, then it is cancelled, logged as delivery-skipped-no-recipient, and surfaced to Nadia as a delivery warning.

**FEAT-11.SPEC-004-AC-08:** Given Priya (Client Reviewer Contact) is a contact on the same client company, when any reminder variant is sent, then Priya is never a recipient on any copy of the email.

**FEAT-11.SPEC-004-AC-09:** Given Owen has no notification preferences that could disable this email, when any reminder is triggered, then it always sends -- there is no opt-out control anywhere in Owen's portal for this email.

**FEAT-11.SPEC-004-AC-10:** Given a day-3 reminder for one invoice and a day-10 reminder for a different invoice both become eligible for the same recipient on the same day, when both fire, then Owen receives two separate emails, each naming its own invoice number -- never a combined email.

**FEAT-11.SPEC-004-AC-11:** Given the Reminder Schedule re-evaluates an invoice whose day-3 Reminder Log entry already has a `sent_at` value, when the re-evaluation runs, then no second day-3 email is sent for that invoice.

**FEAT-11.SPEC-004-AC-12:** Given Dana (Support Operator) has an open support session on the freelancer's account, when she views the invoice's reminder history, then she can see this email's delivery status read-only, and never receives a copy of the email herself.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 3 (day-3, day-10, manual) | 3 |
| Preference States | 1 (always sends -- not configurable) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Bank-Transfer-Pending Reminder Pause

## Overview

**Name:** Bank-Transfer-Pending Reminder Pause
**ID:** FEAT-11.SPEC-005
**Type:** Automation
**Purpose:** Automatically pauses an invoice's reminder schedule while a bank-transfer payment is pending, and resumes it if the pending transfer resolves or reverts, so a reminder never chases a payment that is already on its way.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders

## Scope and Non-Goals

**In Scope:**
- Setting `pause_state` to Paused while bank transfer pending the moment an invoice enters Payment pending status
- Returning `pause_state` to Active when the pending transfer resolves (invoice reaches Paid) or reverts (invoice returns to Overdue/unpaid), unless a freelancer pause is already in effect
- Respecting the precedence rule that a freelancer-initiated pause is never overwritten by this automation, in either direction

**Non-Goals:**
- Deciding what "may send now" means once `pause_state` is set -- owned entirely by FEAT-11.SPEC-002 (Reminder Eligibility Rule); this automation only writes the `pause_state` value, it does not interpret it.
- Detecting or reporting the bank-transfer payment's own status (initiated, pending, succeeded, failed) -- owned by FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync), which this automation only listens to.
- Nadia's own pause/resume action -- owned by FEAT-11.SPEC-003 (Invoice Reminder Panel); this automation never triggers from a freelancer action, only from a payment-status event.
- Card payments -- card payments resolve near-instantly (per FEAT-10.SPEC-003) and never produce a Payment pending status; this automation's trigger fires only for the bank-transfer method, per the dependency map's Invoice status lifecycle.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invoice enters Payment pending status | FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Fires when the payment-processing capability reports a bank-transfer payment as pending confirmation for this invoice | Invoice reference, new status (Payment pending), payment method (bank transfer) |
| Invoice's pending bank transfer resolves or reverts | FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Fires when the payment-processing capability reports the pending transfer as succeeded (invoice reaches Paid) or failed (invoice reverts to its prior unpaid/Overdue status) | Invoice reference, new status (Paid or reverted), payment method (bank transfer) |

## Processing Logic

1. On a Payment pending event for an invoice: read the invoice's current Reminder Log `pause_state`.
2. If `pause_state` is Active, set it to Paused while bank transfer pending.
3. If `pause_state` is already Paused by freelancer, leave it unchanged -- a freelancer-initiated pause is never overwritten by this automation (FEAT-11.SPEC-002's Pause-reason precedence rule).
4. If `pause_state` is already Paused while bank transfer pending (a redundant event, e.g. a duplicate status report), leave it unchanged and take no further action.
5. On a resolve-or-revert event for an invoice: read the invoice's current `pause_state`.
6. If `pause_state` is Paused while bank transfer pending, set it back to Active.
7. If `pause_state` is Paused by freelancer, leave it unchanged -- the freelancer's own pause persists regardless of the payment outcome.
8. If `pause_state` is already Active (a redundant event), take no action.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Paused for pending transfer | Payment pending event arrives while `pause_state` is Active | `pause_state` set to Paused while bank transfer pending | The pause indicator on FEAT-11.SPEC-003 shows "Paused -- bank transfer pending" on next view | FEAT-11.SPEC-002 (Reminder Eligibility Rule), FEAT-11.SPEC-003 (Invoice Reminder Panel) |
| Resumed after resolution | Resolve/revert event arrives while `pause_state` is Paused while bank transfer pending | `pause_state` set to Active | The pause indicator shows "Active" on next view; the invoice becomes eligible for its next due reminder threshold again | FEAT-11.SPEC-002, FEAT-11.SPEC-003 |
| No-action -- freelancer pause preserved | Either event arrives while `pause_state` is Paused by freelancer | None | None -- the indicator continues to show "Paused by you" | -- |
| No-action -- redundant event | Either event arrives while `pause_state` already matches the event's target value | None | None | -- |
| Automation failure | The write to `pause_state` fails (e.g., a transient data-write error) | No change persists | No user-visible interruption; the write is retried automatically | FEAT-13.SPEC-003 (Activity Entry Recording), for logging the retry outcome |

## Data Model

**Reads:** Invoice -- current status, from the Payment pending / resolve-revert event data (FEAT-10.SPEC-004). Reminder Log -- current `pause_state` for the invoice.
**Creates:** None -- this automation never creates a Reminder Log entry, only updates the schedule's live pause state.
**Updates:** Reminder Log -- `pause_state`, set to or cleared from Paused while bank transfer pending.
**Deletes:** None.

## Business Rules

- XBR-15: reminders pause while a bank transfer is pending, per this automation, and resume when it resolves or reverts.
- FEAT-11.SPEC-002's Pause-reason precedence rule governs every write here: this automation may only move `pause_state` between Active and Paused while bank transfer pending; it never sets or clears Paused by freelancer.
- This automation's writes are idempotent per invoice: re-applying the same event twice (e.g., a duplicate status report) never changes an already-correct `pause_state`.
- Processor-reported status is authoritative (dependency map, Invoice Contention note); this automation trusts FEAT-10.SPEC-004's event data without re-verifying the payment status itself.

## Edge Cases

- **Payment pending and resolve/revert events for the same invoice arrive out of order (e.g., a resolve event is processed before its own pending event due to delivery timing)** -- The automation trusts the most recently reported event as authoritative, consistent with FEAT-10.SPEC-004 applying processor-reported status in order; if a resolve event is somehow processed first, the subsequent pending event's step 3/4 check finds `pause_state` already Active or already Paused while bank transfer pending, per the current state, and behaves per the No-action rules rather than producing an inconsistent result.
- **Nadia pauses the invoice herself (FEAT-11.SPEC-003) between the Payment pending event and the resolve event** -- Per the precedence rule, her pause takes priority: the resolve event finds `pause_state` is Paused by freelancer and leaves it unchanged, so the invoice remains paused after the resolve event until Nadia resumes it herself.
- **Concurrent trigger firing (a Payment pending event and a resolve event for the same invoice arrive at effectively the same time, e.g. a very short-lived pending window)** -- Whichever event's write commits first determines the resulting `pause_state`; the second event's read-then-write sees the state left by the first and applies its own step 2-8 logic against that current value, never against a stale read from before the first event committed.
- **A trigger fires for an invoice while a previous run (for the same invoice) is still in flight** -- Writes to `pause_state` for a given invoice are serialized: a second event for the same invoice waits for the in-flight write to commit before reading and applying its own logic, so no write is lost to a race between two events for the same invoice.
- **The pending bank transfer resolves after the invoice has already passed day 10 unpaused** -- If the transfer resolves to Paid, no further reminder is relevant regardless of pause history. If it reverts (fails), `pause_state` returns to Active, but per FEAT-11.SPEC-001, no automatic reminder exists past day 10 -- the invoice's only remaining reminder path is Nadia's manual send (FEAT-11.SPEC-003), now eligible again since `pause_state` is Active.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Triggered by (inbound) | Both the Payment pending event and the resolve/revert event originate here |
| FEAT-11.SPEC-002 (Reminder Eligibility Rule) | Affects (outbound) | Every write to `pause_state` changes what that rule evaluates as eligible |
| FEAT-11.SPEC-003 (Invoice Reminder Panel) | Affects (outbound) | The pause-state indicator reflects this automation's writes |
| FEAT-11.SPEC-001 (Reminder Schedule) | Affects (outbound) | A pause set here can cause the schedule's next eligibility re-check to skip a send |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Affects (outbound) | Automatic pause/resume transitions are logged as part of the invoice's reminder trail |

## Analytics and Success Signals

- **reminder_auto_paused_bank_transfer** (invoice reference) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"
- **reminder_auto_resumed_bank_transfer** (invoice reference, resolution: paid / reverted) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"
- **reminder_auto_pause_write_failed** (retry_count) -- N/A -- no Stage 2 metric measures this automation's internal write failures; retained so a silently failed pause/resume is observable rather than invisible, and surfaced through FEAT-13's trail rather than a dedicated success metric.

## Acceptance Criteria

**FEAT-11.SPEC-005-AC-01:** Given Nadia's invoice to Owen has an Active reminder schedule, when a bank-transfer payment for that invoice enters Payment pending, then `pause_state` becomes Paused while bank transfer pending.

**FEAT-11.SPEC-005-AC-02:** Given the invoice's `pause_state` is Paused while bank transfer pending, when the transfer resolves to Paid, then `pause_state` returns to Active.

**FEAT-11.SPEC-005-AC-03:** Given the invoice's `pause_state` is Paused while bank transfer pending, when the transfer fails and the invoice reverts to unpaid, then `pause_state` returns to Active and the invoice becomes eligible for its next due reminder.

**FEAT-11.SPEC-005-AC-04:** Given Nadia has already paused the invoice herself (Paused by freelancer), when a bank-transfer payment for it enters Payment pending, then `pause_state` remains Paused by freelancer, unchanged.

**FEAT-11.SPEC-005-AC-05:** Given the invoice's `pause_state` is Paused by freelancer, when the pending transfer that triggered no automatic pause resolves, then `pause_state` remains Paused by freelancer -- the resolve event never touches it.

**FEAT-11.SPEC-005-AC-06:** Given the invoice's `pause_state` is already Paused while bank transfer pending, when a duplicate Payment pending event is reported for the same invoice, then `pause_state` remains unchanged and no error occurs.

**FEAT-11.SPEC-005-AC-07:** Given the invoice's `pause_state` is already Active, when a duplicate resolve event is reported, then `pause_state` remains unchanged.

**FEAT-11.SPEC-005-AC-08:** Given a Payment pending event and a resolve event for the same invoice arrive at effectively the same time, when both are processed, then the resulting `pause_state` reflects whichever event's write committed first, with the second event's logic applied against that resulting state rather than a stale read.

**FEAT-11.SPEC-005-AC-09:** Given the write to `pause_state` fails on the first attempt, when the automation retries and the retry succeeds, then `pause_state` reflects the correct value with no user-visible interruption.

**FEAT-11.SPEC-005-AC-10:** Given a pending bank transfer for an invoice already past day 10 reverts to unpaid, when `pause_state` returns to Active, then no automatic reminder is scheduled (per FEAT-11.SPEC-001's day-10 ceiling), and only Nadia's manual send remains available.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (Payment pending, resolve/revert) | 2 |
| Outcome Paths | 5 (paused, resumed, freelancer-pause preserved, redundant no-action, automation failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
