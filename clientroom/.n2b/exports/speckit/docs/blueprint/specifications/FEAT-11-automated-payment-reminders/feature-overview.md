---
document_type: feature-overview
feature_number: FEAT-11
feature_name: Automated Payment Reminders
feature_slug: automated-payment-reminders
priority_tier: Core
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 5
screen_count: 1
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 1
---

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
