---
document_type: feature-overview
feature_number: FEAT-09
feature_name: Invoice Generation & Sending
feature_slug: invoice-generation-sending
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 10
screen_count: 3
automation_count: 2
logic_rule_count: 4
integration_count: 0
notification_count: 1
---

# Feature Breakdown Brief: Invoice Generation & Sending

## Summary

**Feature:** Invoice Generation & Sending
**ID:** FEAT-09
**Description:** An invoice is generated whenever a deposit is accepted, a milestone is approved, or a project is marked complete, and sent to the client's Primary Contact by email with a pay link.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Problem Statement: invoices today "live in a spreadsheet plus a pasted payment link that gets chased for weeks" — automatic, correctly-triggered invoicing is a primary reason the product exists. MVP phase: without it, approvals and deposits have no financial consequence. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Automatic invoicing — a deposit, milestone approval, or project completion triggers the right invoice
- Ad-hoc invoicing — freelancer can issue an invoice manually outside the schedule
- Send with a pay link — the client receives the invoice with a direct way to pay
- Payment terms and due dates — each invoice carries a due date from Nadia's default payment terms, adjustable before sending
- Download a copy — Nadia and the Primary Contact can save a printable copy of any invoice, credit note, or payment receipt

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-09.SPEC-001 | Invoice List | Screen | Nadia (Freelancer), Owen (Client Primary Contact) | Nadia browses a project's invoices and Owen browses his own; both see the "no invoices issued" empty state before the first one exists |
| FEAT-09.SPEC-002 | Invoice Detail | Screen | Nadia (Freelancer), Owen (Client Primary Contact) | Shows one invoice's amount, status, due date, reminder history, and pay-link/download controls, role-differentiated by what each side may do |
| FEAT-09.SPEC-003 | Manual Invoice & Credit Note Issuance | Screen | Nadia (Freelancer) | Nadia issues an ad-hoc invoice outside the schedule, or a credit note correcting a previously sent invoice, with an adjustable due date |
| FEAT-09.SPEC-004 | Automatic Invoice Generation | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | On a deposit acceptance, milestone approval, or project completion, generates the correct invoice with no freelancer action and hands it off to be sent |
| FEAT-09.SPEC-005 | Manual Invoice & Credit Note Recording | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | Validates and writes an ad-hoc invoice or a credit note submitted from SPEC-003, marking the original invoice Corrected when a credit note supersedes it |
| FEAT-09.SPEC-006 | Invoice Access & Role Authorization Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Governs who may view, issue, pay-link-follow, or download an invoice, including Priya's total exclusion and Dana's read-only support-session visibility |
| FEAT-09.SPEC-007 | Invoice Content, Numbering, Amount & Due-Date Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact) | Governs sequential numbering, required business/billing details, amount-matches-trigger validation, and default-payment-terms due-date derivation for every invoice |
| FEAT-09.SPEC-008 | Invoice Immutability & Correction Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact) | Enforces that a sent invoice is never silently edited and that a correction is always a visible credit note or a new invoice |
| FEAT-09.SPEC-009 | Pay-Link Availability & No-Account Fallback Rule | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact) | Governs what an invoice's pay link shows and how it is worded when Nadia has no connected, ready payment account |
| FEAT-09.SPEC-010 | Invoice Issued & Copy Confirmation Notification | Notification | Owen (Client Primary Contact), Nadia (Freelancer) | Emails Owen the invoice with its pay link and emails Nadia a confirmation copy, whenever any invoice (automatic, ad hoc, or credit note) is sent |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Automatic invoicing — a deposit, milestone approval, or project completion triggers the right invoice | FEAT-09.SPEC-004, FEAT-09.SPEC-007, FEAT-09.SPEC-010 | The automation fires on each of the three triggers with no freelancer action; the content/numbering rules populate the invoice; the notification sends it | Phase 2 (Explicit) |
| Ad-hoc invoicing — freelancer can issue an invoice manually outside the schedule | FEAT-09.SPEC-003, FEAT-09.SPEC-005 | The screen gives Nadia the manual-issuance form; the recording automation validates and writes it | Phase 2 (Explicit) |
| Send with a pay link — the client receives the invoice with a direct way to pay | FEAT-09.SPEC-002, FEAT-09.SPEC-009, FEAT-09.SPEC-010 | The detail screen displays the pay link; the availability rule governs its wording when payment is not yet connectable; the notification carries it in Owen's email | Phase 2 (Explicit) |
| Payment terms and due dates — each invoice carries a due date from Nadia's default payment terms, adjustable before sending | FEAT-09.SPEC-007, FEAT-09.SPEC-003 | The rule spec derives the due date from `default_payment_terms`; the manual-issuance screen exposes it as an adjustable field before sending | Phase 2 (Explicit) |
| Download a copy — Nadia and the Primary Contact can save a printable copy of any invoice, credit note, or payment receipt | FEAT-09.SPEC-002 | Download control on the invoice detail screen, available to both Nadia and Owen for any invoice, credit note, or receipt | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-09.SPEC-001 | Invoice List | Phase 3 (Entity-Lifecycle Analysis) | The Invoice entity's CRUD matrix has no Read (list) coverage without a dedicated list, and the feature's own States field names an explicit empty state ("no invoices issued") that needs a screen home |
| FEAT-09.SPEC-006 | Invoice Access & Role Authorization Rules | Phase 5 (Rule-Constraint Discovery) | The Access field's four fully differentiated role behaviors (Nadia Full, Owen Own-only, Priya None, Dana View-in-session), combined with the invoice's GDPR-class data sensitivity, exceed the inline-authorization threshold and govern three separate screens |
| FEAT-09.SPEC-008 | Invoice Immutability & Correction Rules | Phase 5 (Rule-Constraint Discovery) | The Primary Flows & Alternates' correction line and the entity dependency map's "never edited after sending" lifecycle note together form a rule shared across SPEC-002, SPEC-003, and SPEC-005 that is under-specified if left inline |
| FEAT-09.SPEC-009 | Pay-Link Availability & No-Account Fallback Rule | Phase 4 (External Dependencies lens) | The "no payment account connected yet" alternate flow depends on an external capability (payment processing, ASMP-28) owned by another feature's future Integration spec (FEAT-32); this feature's own rule for how that dependency surfaces on its invoices needed a standalone spec |

## Entity-Lifecycle Coverage Matrix

**Entity: Invoice** *(this feature owns creation, numbering, and the send step; later status changes belong to other features)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-09.SPEC-004, FEAT-09.SPEC-005 | SPEC-004 creates the invoice automatically on a deposit/approval/completion trigger; SPEC-005 creates an ad-hoc invoice or a credit note from Nadia's manual submission | Both apply the numbering/content rules in FEAT-09.SPEC-007 |
| Read (single) | FEAT-09.SPEC-002 | Invoice Detail loads one invoice's amount, status, due date, reminder history, and pay-link state | Reminder history is read from the Reminder Log (dependency map: Reminder Log read by FEAT-09) |
| Read (list) | FEAT-09.SPEC-001 | Invoice List shows every invoice for a project (Nadia) or for a client company (Owen), including the "no invoices issued" empty state | — |
| Update | FEAT-09.SPEC-003, FEAT-09.SPEC-004, FEAT-09.SPEC-005 | SPEC-003/SPEC-004 set `due_date` before sending (adjustable); SPEC-004/SPEC-005 set `status: Sent` once the send is attempted; SPEC-005 sets the original invoice's `status: Corrected` when a credit note supersedes it | All other status transitions — Payment pending, Paid, Overdue, Refunded, Partially refunded, Disputed — are owned by FEAT-10, FEAT-11, and FEAT-25, not this feature |
| Delete/Archive | N/A | This feature never deletes or archives an Invoice. Deletion is owned exclusively by account deletion (FEAT-24), subject to legal financial-record retention (scope-boundaries.md, SC-24) — an explicit non-goal recorded below, not a silent omission | — |
| State Transition | FEAT-09.SPEC-004, FEAT-09.SPEC-005 (→ Sent); FEAT-09.SPEC-005 (original invoice → Corrected) | Generated → Sent is this feature's own transition; Corrected is written on the original invoice the moment a credit note is issued against it | Payment pending → Paid → Overdue → Refunded/Disputed transitions belong to FEAT-10, FEAT-11, and FEAT-25 |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Project | FEAT-09.SPEC-001, FEAT-09.SPEC-002, FEAT-09.SPEC-003, FEAT-09.SPEC-004 | Identifies which project an invoice belongs to and supplies the completion trigger event |
| Client | FEAT-09.SPEC-003, FEAT-09.SPEC-004, FEAT-09.SPEC-007 | Supplies the client's billing name and address printed on every invoice (FEAT-01, XBR-16) |
| Client Contact | FEAT-09.SPEC-001, FEAT-09.SPEC-002, FEAT-09.SPEC-006 | Identifies Owen as the invoice's recipient and enforces the Primary-vs-Reviewer access gate |
| Milestone | FEAT-09.SPEC-004 | The approval-triggered generation path reads which milestone approved and its price | — |
| Payment Schedule | FEAT-09.SPEC-004 | Read at the moment of each trigger to determine the correct invoice amount and whether that trigger fires an invoice at all (XBR-01/02/03) |
| Branding Profile | FEAT-09.SPEC-002, FEAT-09.SPEC-010 | The freelancer's logo and brand colour apply to the invoice detail screen and the invoice email (XBR-31) | — |
| Freelancer Account | FEAT-09.SPEC-007 | Supplies business name, business address, tax ID, and `default_payment_terms` printed on and used to compute every invoice (XBR-16) |
| Payment Account Connection | FEAT-09.SPEC-002, FEAT-09.SPEC-009 | Supplies pay-link availability status so the detail screen and the availability rule can render the correct wording (XBR-19) |
| Reminder Log | FEAT-09.SPEC-002 | Invoice Detail displays reminder history (day 3, day 10, manual, pause state) alongside the invoice itself |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| A proposal acceptance is recorded and the schedule includes a deposit | Generate the deposit invoice with correct amount, currency, and tax line | Standalone Automation | FEAT-09.SPEC-004 |
| A milestone approval is recorded | Generate the next invoice in the payment schedule, with no freelancer action | Standalone Automation | FEAT-09.SPEC-004 |
| A project is marked complete and the schedule includes an on-completion payment | Generate the on-completion invoice | Standalone Automation | FEAT-09.SPEC-004 |
| An invoice is generated by any of the three automatic triggers | Assign a unique sequential number, business/billing details, issue date, and a due date from Nadia's default payment terms | Standalone Logic/Rule (applied inline within SPEC-004) | FEAT-09.SPEC-007 |
| An invoice is ready to leave Generated state | Email Owen the invoice with its pay link; email Nadia a confirmation copy | Standalone Notification | FEAT-09.SPEC-010 |
| An invoice email send fails | Retry the send without creating a duplicate invoice record | Inline in SPEC-010 (Error state), enforced by SPEC-004/SPEC-005's exactly-once creation | FEAT-09.SPEC-010 |
| Nadia issues an invoice outside the schedule (e.g., a scope addition) | Validate and record the ad-hoc invoice with the same numbering/due-date/content rules as an automatic one | Standalone Automation | FEAT-09.SPEC-005 |
| Nadia issues a credit note against a sent invoice | Record the credit note as a new, linked Invoice; mark the original invoice Corrected; never alter the original record | Standalone Automation, governed by Logic/Rule | FEAT-09.SPEC-005 / FEAT-09.SPEC-008 |
| An invoice is generated but Nadia has no connected payment account | Issue and send it anyway, stating online payment is not yet available and how to pay Nadia directly; prompt Nadia to connect her account | Standalone Logic/Rule | FEAT-09.SPEC-009 |
| A previously working Payment Account Connection later needs attention or disconnects | Pay links on open invoices switch to "online payment is temporarily unavailable" | Standalone Logic/Rule (inbound event from FEAT-32) | FEAT-09.SPEC-009 |
| Nadia or Owen opens an invoice and taps Download | Produce a printable copy of the invoice, credit note, or receipt | Inline in triggering screen | FEAT-09.SPEC-002 |
| An invoice is generated, sent, manually issued, corrected, or downloaded | Emit the corresponding signal (`invoice_generated`, `invoice_sent`, `invoice_manually_issued`, `invoice_correction_issued`, `invoice_copy_downloaded`) | Inline in triggering spec | FEAT-09.SPEC-004 / SPEC-005 / SPEC-010 / SPEC-002 |
| An invoice is generated, sent, or corrected | Write an append-only Activity Log Entry (XBR-05) | Cross-feature — owned by Immutable Activity & Audit Trail (FEAT-13) | FEAT-13 responsibility |
| Priya opens any invoice-related view | Show no invoice content at all, per her Reviewer role | Standalone Logic/Rule | FEAT-09.SPEC-006 |
| Dana opens an invoice inside a support session | Show read-only invoice content with no send, download, pay, or issue control, inside the logged session | Cross-feature (FEAT-31), governed inline by SPEC-006 | FEAT-09.SPEC-006 |
| A project has no invoices yet | Show "no invoices issued" | Inline in triggering screen (Empty state) | FEAT-09.SPEC-001 |
| Nadia attempts a manual issuance or Owen opens an invoice detail page while offline | Show a clear "reconnect" state; the action never appears to succeed, and the last-known invoice detail is shown as possibly out of date if cached | Inline in triggering screens (Offline/Degraded state) | FEAT-09.SPEC-002 / FEAT-09.SPEC-003 |

## Shared Context

**Shared Entities:**
- Invoice — created by SPEC-004 (automatic) and SPEC-005 (manual/credit note); read singly by SPEC-002, listed by SPEC-001; content/numbering governed by SPEC-007; immutability and correction governed by SPEC-008; pay-link display governed by SPEC-009. Fields touched: `invoice_number`, `project`, `triggering_event`, `amount`, `currency`, `tax_label`, `tax_rate`, `total`, business/billing details, `issue_date`, `due_date`, `status` (Generated, Sent, Corrected within this feature's scope), `pay_link availability`.
- Payment Schedule, Freelancer Account, Client — read-only sources for the amount, due date, and content every invoice carries (SPEC-004, SPEC-005, SPEC-007); never written by this feature.
- Payment Account Connection — read-only source for pay-link availability wording on SPEC-002 and SPEC-009; a status change here (connect, needs attention, disconnect) is an inbound event this feature reacts to but never causes.

**Shared UI Patterns:**
- Invoice summary card — the amount, status badge, due date, and download control appear identically on SPEC-001 (as a list row) and SPEC-002 (as the full detail); Spec Writers should describe one visual pattern used at two densities rather than two unrelated layouts.
- "Not yet edited after sending" convention — once an invoice's status is Sent or later, SPEC-002 replaces any would-be edit control with a plain record display and a "Correct with a credit note" action; SPEC-003 reuses this same convention when Nadia opens the correction flow from SPEC-002.
- Pay-link status banner — SPEC-002 and SPEC-010's email both render the same three pay-link states (ready to pay, online payment not yet available, temporarily unavailable) with identical wording, sourced from SPEC-009.

**Shared Validation:**
- SPEC-007 defines the single source of truth for numbering, required content, amount-matches-trigger validation, and due-date derivation. SPEC-004 and SPEC-005 both apply it at creation time rather than re-deriving these rules.
- SPEC-006 defines the single source of truth for who may view, issue, or act on an invoice. SPEC-001, SPEC-002, and SPEC-003 all reference it rather than restating role gates.
- SPEC-008 defines the single source of truth for immutability and correction. SPEC-002 (no edit control after send), SPEC-003 (correction entry point), and SPEC-005 (correction recording) all reference it.

## Internal Dependency Map

```
FEAT-09.SPEC-001 (Invoice List) -> [Nadia or Owen selects an invoice] -> FEAT-09.SPEC-002 (Invoice Detail)
FEAT-09.SPEC-001 (Invoice List) -> [Nadia taps "New Invoice"] -> FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance)
FEAT-09.SPEC-002 (Invoice Detail) -> [Nadia taps "Issue Credit Note"] -> FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance)
FEAT-03 (Proposal Acceptance) -> [deposit due] -> FEAT-09.SPEC-004 (Automatic Invoice Generation) [cross-feature trigger]
FEAT-08 (Milestone Approval) -> [approval recorded] -> FEAT-09.SPEC-004 (Automatic Invoice Generation) [cross-feature trigger]
FEAT-01 (Client & Project Management) -> [project marked complete] -> FEAT-09.SPEC-004 (Automatic Invoice Generation) [cross-feature trigger]
FEAT-09.SPEC-004 (Automatic Invoice Generation) -> [applies] -> FEAT-09.SPEC-007 (Content, Numbering, Amount & Due-Date Rules)
FEAT-09.SPEC-004 (Automatic Invoice Generation) -> [checks] -> FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule)
FEAT-09.SPEC-004 (Automatic Invoice Generation) -> [invoice ready] -> FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification)
FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) -> [Nadia submits] -> FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording)
FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording) -> [applies] -> FEAT-09.SPEC-007 (Content, Numbering, Amount & Due-Date Rules)
FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording) -> [correction path] -> FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules)
FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording) -> [invoice ready] -> FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification)
FEAT-09.SPEC-001 (Invoice List) -> [validates access using] -> FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules)
FEAT-09.SPEC-002 (Invoice Detail) -> [validates access using] -> FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules)
FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) -> [validates access using] -> FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules)
FEAT-09.SPEC-002 (Invoice Detail) -> [renders pay-link status governed by] -> FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule)
FEAT-09.SPEC-002 (Invoice Detail) -> [no edit control once Sent, per] -> FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules)
```

**Default Entry:** FEAT-09.SPEC-001 (Invoice List) for Nadia, reached from a project's invoices area (FEAT-01) or the Financial Dashboard's client drill-down (FEAT-12); Owen's default entry is FEAT-09.SPEC-002 (Invoice Detail), reached directly from the invoice email link (FEAT-09.SPEC-010) or from his own portal's invoice list (FEAT-09.SPEC-001).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-09.SPEC-004 | Inbound | FEAT-03 (Proposal Acceptance) | Acceptance fires the deposit-invoice trigger (XBR-01), using the schedule as it stood at acceptance | Acceptance recorded with a deposit in the schedule |
| FEAT-09.SPEC-004 | Inbound | FEAT-08 (Milestone Approval) | Approval fires the next-invoice trigger (XBR-02), with no freelancer action | Milestone approval recorded |
| FEAT-09.SPEC-004 | Inbound | FEAT-01 (Client & Project Management) | Marking a project complete fires the on-completion trigger (XBR-03) | Project marked complete |
| FEAT-09.SPEC-004, FEAT-09.SPEC-007 | Inbound | FEAT-15 (Currency & Tax Handling) | Reads the project's configured currency and tax label/rate to compute the tax line and total (XBR-17); FEAT-15 owns the derivation, this feature only applies it | Invoice generated |
| FEAT-09.SPEC-007 | Inbound | FEAT-21 (Settings & Account Management) | Reads the freelancer's business details and `default_payment_terms` to populate every invoice and compute its due date (XBR-16) | Invoice generated or manually issued |
| FEAT-09.SPEC-007 | Inbound | FEAT-01 (Client & Project Management) | Reads the client's billing name and address (XBR-16); sending is blocked until both this and the freelancer's business details exist | Invoice generated or manually issued |
| FEAT-09.SPEC-002, FEAT-09.SPEC-009 | Inbound | FEAT-32 (Payment Account Connection) | Pay-link availability and its wording are derived from the connection's status (XBR-19); FEAT-32 owns the connection itself | Connection connected, needs attention, or disconnected |
| FEAT-09.SPEC-002 | Outbound | FEAT-10 (Invoice Payment Processing) | Owen follows the pay link to pay; Nadia records an off-platform payment from the same detail screen | Pay link followed, or Nadia marks paid elsewhere |
| FEAT-09.SPEC-002 | Outbound | FEAT-25 (Refunds, Reversals & Cancellations) | Nadia records a refund or partial refund against a sent invoice from its detail view | Nadia records a refund issued through her processor |
| FEAT-09.SPEC-004, FEAT-09.SPEC-005 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Every generation, send, manual issuance, and correction writes an append-only trail entry (XBR-05) | Invoice generated, sent, or corrected |
| FEAT-09.SPEC-010 | Outbound | FEAT-14 (Notifications & Delivery) | Relies on the product's transactional email delivery capability for the actual send and delivery/bounce status; this feature carries no Integration spec of its own for that capability | Notification queued |
| FEAT-09 (feature-level) | Outbound | FEAT-11 (Automated Payment Reminders) | The due date this feature sets on every invoice is what FEAT-11's reminder schedule counts from (XBR-16) | Invoice sent |
| FEAT-09 (feature-level) | Outbound | FEAT-12 (Freelancer Financial Dashboard) | Invoice amounts, currency, and status feed the dashboard's earned/outstanding/overdue totals (XBR-22) | Invoice created or its status changes |
| FEAT-09.SPEC-001, FEAT-09.SPEC-002 | Inbound | FEAT-12 (Freelancer Financial Dashboard) | Client drill-down navigates into this feature's list and detail views | Nadia opens a specific client's invoices |
| FEAT-09.SPEC-001, FEAT-09.SPEC-003 | Inbound | FEAT-01 (Client & Project Management) | The project view's invoices area is the entry point into this feature's list and manual-issuance screens | Nadia opens the invoices area or issues an ad-hoc invoice |
| FEAT-09.SPEC-002 | Inbound | FEAT-31 (Operator Support Access) | Dana views invoice detail read-only, inside a logged, time-limited support session | Support session opened |
| FEAT-09 (feature-level) | Outbound | FEAT-22 (Accounting Export) | Invoice records are a required input to the generated accounting export file | Export generated |
| FEAT-09 (feature-level) | Outbound | FEAT-28 (Search) | Invoices are indexed so Nadia can find one by search result | Search index updated |

## Non-Functional Notes

**Data volumes / growth:** Invoice volume tracks each project's payment schedule — typically a handful of invoices per project across a freelancer's 3–15 active clients (assumptions-constraints.md, ASMP-22) — with no meaningful scale concern of its own; sent invoices are retained for the life of the freelancer's account (scope-boundaries.md, SC-24).

**Responsiveness:** Automatic invoice generation is near-instant, with no perceptible loading state between the triggering event and the invoice appearing (feature's own States field); the client-facing invoice detail and pay-link screens become interactive within roughly 2 seconds on a typical mobile connection (assumptions-constraints.md, ASMP-21/ASMP-27).

**Data sensitivity / privacy:** Invoices are financial records carrying personal data — client billing name and address, and the recipient contact's identity — treated as GDPR-class personal data (assumptions-constraints.md, ASMP-24); they are evidentiary and immutable once sent (assumptions-constraints.md, ASMP-25; feature-dependency-map.md, Entity: Invoice, Data Sensitivity); no card or bank data is ever captured or stored by the product itself (assumptions-constraints.md, ASMP-24; scope-boundaries.md, SC-10). Invoice content is hidden entirely from Reviewer contacts (Access field).

**Compliance flags:** Every invoice carries the content commonly required of a valid invoice worldwide — a unique sequential number, both parties' business details, an issue date, a due date, and a tax line (assumptions-constraints.md, ASMP-24); sent invoices are retained for as long as the freelancer's account exists and may be subject to a legal financial-record retention period even after account deletion (scope-boundaries.md, SC-24; XBR-33).

## Non-Goals

- **Automatic tax calculation** — Excluded per scope-boundaries.md (SC-16): every invoice's tax line is a freelancer-configured label and rate owned by Currency & Tax Handling (FEAT-15); this feature applies that configuration but performs no tax computation of its own.
- **Recurring or retainer billing on a fixed calendar schedule** — Excluded per scope-boundaries.md (SC-14): billing is deposit, per-milestone, or on-completion, driven entirely by the project's Payment Schedule; an occasional retainer charge is issued through this feature's own ad-hoc invoicing capability instead of a calendar-based recurrence engine.
- **Partial payments or instalments on a single invoice** — Excluded per scope-boundaries.md (SC-17): an invoice is either unpaid or paid in full, owned by Invoice Payment Processing (FEAT-10); instalments are expressed as separate milestone or deposit invoices this feature already generates through the payment schedule.
- **Issuing refunds or fighting chargebacks inside this feature** — Excluded per scope-boundaries.md (SC-18): the platform never holds or moves funds; a refund is issued by Nadia through her own processor account and only recorded against the invoice by Refunds, Reversals & Cancellations (FEAT-25), reached from this feature's invoice detail screen.
- **A configurable invoicing workflow or automation builder** — Excluded per scope-boundaries.md (SC-11): this feature ships fixed, sensible auto-invoice behavior (deposit, approval, completion) rather than a configurable trigger or approval-routing builder, in line with the brief's "the freelancer stops chasing" promise.
- **A standalone Integration spec for payment processing or transactional email in this feature** — The payment-processing capability behind the pay link is owned by Payment Account Connection (FEAT-32, XBR-19's stated authority); the transactional email capability behind every send is owned by Notifications & Delivery (FEAT-14, per the External Touchpoints table). Both are recorded above as cross-feature touchpoints rather than duplicated as Integration specs here, matching the disposition already used by FEAT-03 and FEAT-08 for the same shared capabilities.
- **Bulk import of historical invoices from another tool** — Excluded per scope-boundaries.md (SC-19): imported invoices were never generated or sent through Clientroom and so could not carry this feature's evidence guarantee (immutability, sequential numbering); every invoice in the product is created through SPEC-004 or SPEC-005.
- **Retention/purge policy for sent invoices** — Intentional lifecycle decision confirmed by the CRUD matrix: sent invoices are retained for the life of the freelancer's account with no automatic purge, per scope-boundaries.md (SC-24); they are removed only on account deletion (FEAT-24), subject to legal financial-record retention (XBR-33). This feature makes no delete or archive decision of its own.
