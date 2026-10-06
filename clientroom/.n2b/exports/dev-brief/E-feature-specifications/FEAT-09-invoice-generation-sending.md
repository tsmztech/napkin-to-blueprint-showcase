# FEAT-09 — Invoice Generation & Sending

This chapter covers Invoice Generation & Sending, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 10 specifications carrying 131 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-09.SPEC-001 | Invoice List | screen | 11 |
| FEAT-09.SPEC-002 | Invoice Detail | screen | 14 |
| FEAT-09.SPEC-003 | Manual Invoice & Credit Note Issuance | screen | 13 |
| FEAT-09.SPEC-004 | Automatic Invoice Generation | automation | 13 |
| FEAT-09.SPEC-005 | Manual Invoice & Credit Note Recording | automation | 12 |
| FEAT-09.SPEC-006 | Invoice Access & Role Authorization Rules | logic-rule | 15 |
| FEAT-09.SPEC-007 | Invoice Content, Numbering, Amount & Due-Date Rules | logic-rule | 16 |
| FEAT-09.SPEC-008 | Invoice Immutability & Correction Rules | logic-rule | 12 |
| FEAT-09.SPEC-009 | Pay-Link Availability & No-Account Fallback Rule | logic-rule | 12 |
| FEAT-09.SPEC-010 | Invoice Issued & Copy Confirmation Notification | notification | 13 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Invoice List

## Overview

**Name:** Invoice List
**ID:** FEAT-09.SPEC-001
**Type:** Screen
**Purpose:** Lets Nadia browse a project's invoices and lets Owen browse his own company's invoices, each scoped to what their role can see, with a clear "no invoices issued" state before the first one exists.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- Nadia's per-project invoice list, reached from Project Detail (FEAT-01.SPEC-005)
- Owen's own-company invoice list, scoped across all of his company's projects with one freelancer
- The empty state shown before a project's or company's first invoice exists
- Selecting an invoice to open its detail (FEAT-09.SPEC-002)
- Nadia's entry point into issuing an ad-hoc invoice or credit note (FEAT-09.SPEC-003)

**Non-Goals:**
- Invoice detail content (amount, status, reminder history, pay link, download) -- owned entirely by FEAT-09.SPEC-002 (Invoice Detail); this screen shows only summary rows
- Issuing or recording an ad-hoc invoice or credit note -- this screen only provides the entry point; the form and recording logic belong to FEAT-09.SPEC-003 and FEAT-09.SPEC-005
- Cross-client search across a freelancer's whole roster -- excluded per scope-boundaries.md's Deferral Notes (Global Search Across Clients & Projects is a v1-phase feature, FEAT-28); this screen lists only the invoices of one project (Nadia) or one client company (Owen)
- Financial totals or aggregation across invoices -- owned by Freelancer Financial Dashboard (FEAT-12); this screen lists individual invoice rows, not sums

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-005 (Project Detail) | Nadia opens the project's invoices area | Project reference -- list scoped to that project's invoices |
| FEAT-12 (Freelancer Financial Dashboard) | Nadia drills into a client from the dashboard | Client reference -- list scoped to all of that client's invoices across its projects |
| FEAT-05.SPEC-003 (Portal Home) | Owen opens "Invoices" from his portal home | Client Contact identity -- list scoped to his own company's invoices, own-only |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Owen taps "View all invoices" from an invoice email | Client Contact identity -- same own-company scope as the portal entry |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full list for the scoped project or client -- every invoice regardless of status | Open any row; issue a new ad-hoc invoice or credit note | -- |
| Owen (Client Primary Contact) | Full list of his own company's invoices, own-only, across all of that company's projects with this freelancer | Open any of his own company's invoice rows | -- |
| Priya (Client Reviewer Contact) | None -- per FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules), invoice content is hidden entirely from Reviewer contacts | None | The "Invoices" entry point is not shown anywhere in Priya's portal navigation; a direct link is redirected to her portal home with no error message, consistent with FEAT-09.SPEC-006 |
| Dana (Support Operator) | Full list, read-only, inside a logged support session (FEAT-31) | View only -- no row action creates, sends, or downloads anything | The "New Invoice" action is not rendered for Dana; selecting a row opens FEAT-09.SPEC-002 in its own read-only mode |
| Unauthenticated | No | No | Redirected to FEAT-05.SPEC-001 (Request Sign-In Link) if a client contact, or to sign-in if Nadia; no invoice data is ever rendered before authentication |
| Expired session | No | No | Client contact: FEAT-05.SPEC-002's expired-link explanation with a one-tap fresh-link request. Nadia: redirected to sign-in; the list's scroll position is not preserved across re-authentication |

## Layout and Content

**Header:** Screen title -- "Invoices" for Nadia (with the project or client name shown beneath it, depending on entry context) or "Your invoices" for Owen. Nadia's header includes a "New Invoice" action (top-right) that opens FEAT-09.SPEC-003.

**Body:** A vertical list of invoice summary card rows (the shared pattern also used at full density on FEAT-09.SPEC-002), each row showing: invoice number, the triggering event's short label (Deposit, Milestone: {name}, On completion, Ad hoc, Credit note), amount and currency, a status badge, and the due date. Rows are ordered newest-issued first. Nadia's list additionally shows the project or client's name column when scoped from the dashboard (multi-project context); Owen's list shows which project each invoice belongs to, since his scope spans every project with this freelancer.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Rows stack their contents vertically (invoice number and status badge on the first line, amount/due date on the second); the list scrolls independently of the header.
- **Medium size class and above:** Rows lay out horizontally in a single line (number, project/client, amount, status, due date), capped at a consistent platform-wide content width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Invoice row | Tap | Navigate to FEAT-09.SPEC-002 (Invoice Detail) for that invoice | Screen transitions to detail | Standard navigation transition |
| "New Invoice" action (Nadia only) | Tap | Navigate to FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | Screen transitions to the issuance form | Standard navigation transition |
| Status badge | Display only | None | None | Communicates status by label and icon, never colour alone |
| List (on reopen) | Screen regains focus after being backgrounded | Re-fetches the current list | Rows refresh to current status | Rows update silently if unchanged; a changed status shows briefly highlighted |

### Accessibility Notes

- **Focus order:** Header title -> "New Invoice" action (Nadia only) -> invoice rows in display order.
- **Status announcements:** A status badge that changes while the list is open (e.g., an invoice becomes Overdue) is announced to assistive technology as "{invoice number} is now {status}."
- **Keyboard alternatives:** Every row and action is reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | "No invoices issued" message, with a "New Invoice" action for Nadia (Owen and Dana see the message with no action) | The scoped project or client has zero invoices | The first invoice is generated or issued |
| Loading | Skeleton rows in place of invoice cards | Screen first opens, or scope changes | Data finishes loading |
| Populated | Rows as described in Layout and Content | Loading completes with at least one invoice | Scope changes or screen closes |
| Error | Error banner "Couldn't load invoices. Check your connection and try again." with Retry | The list fails to load | User taps Retry and the load succeeds |
| Offline/Degraded | A "You're offline -- reconnect to see the latest invoices" banner sits above the last successfully loaded rows, which remain visible but are marked as possibly out of date | Connectivity is lost while the screen is open or being opened | Connectivity returns and the list re-fetches silently |

## Validation Rules

Not applicable -- this screen has no user input beyond navigation and selection.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Invoice row tap | FEAT-09.SPEC-002 (Invoice Detail) | -- |
| "New Invoice" tap (Nadia) | FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | -- |

## Data Model

**Creates:** None.
**Reads:** Invoice -- `invoice_number`, `project`, `triggering_event`, `amount`, `currency`, `total`, `status`, `due_date`, scoped to the current Project (Nadia's project entry) or Client (Nadia's dashboard drill-down and Owen's own-company scope).
**Updates:** None.
**Deletes:** None.

## Business Rules

- FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules) governs which roles reach this screen at all and what each sees; this screen's Access and Visibility table is consistent with that spec.
- Owen's scope is Own-only across every project his company has with this freelancer (per the Access Matrix), not limited to a single project, since a client contact's relationship spans the whole client company.
- The empty state is functionally identical whether reached from a brand-new project or a brand-new client -- "no invoices issued" never distinguishes the reason.

## Edge Cases

- **Nadia's project has invoices but none match the current status filter (if one is applied elsewhere in the UI)** -- Not applicable: this screen defines no filter of its own; every invoice in scope is always shown.
- **A new invoice is generated automatically while the list is open** -- The list re-fetches on the same silent-refresh basis as any status change (per Interactions) and the new row appears without the user reloading the screen manually.
- **Owen's company has invoices across three different projects** -- All are shown in one list, each row carrying its own project label, since his scope is company-wide rather than per-project.
- **The scoped project or client is archived after invoices were issued** -- The list remains reachable and fully populated; archiving a project (FEAT-01) does not remove or hide its invoice history.
- **Two of Nadia's sessions view the same project's invoice list while a third session issues a new invoice** -- No conflict: this is a read-only list with no write path of its own, so both sessions simply refresh to show the new row; there is no concurrent-edit conflict to resolve here, since a list never sets `status` or any other Invoice field. Editing an individual invoice's contention is handled entirely on FEAT-09.SPEC-002.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (outbound) | Selecting a row opens that invoice's detail |
| FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | Navigation (outbound) | Nadia's "New Invoice" action opens the issuance form |
| FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules) | References (inbound) | Governs the Access and Visibility table above |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (inbound) | Nadia arrives from a project's invoices area |
| FEAT-05.SPEC-003 (Portal Home) | Navigation (inbound) | Owen arrives from his portal home |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Navigation (inbound) | Owen arrives from an invoice email's "View all invoices" link |
| FEAT-12 (Freelancer Financial Dashboard) | Navigation (inbound) | Nadia arrives from a client drill-down |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| invoice_list_viewed | scope (project / client / own-company), invoice count, viewer role | List finishes loading with a populated or empty state | N/A -- no success-metrics.md metric tracks browsing this list directly; retained as the entry-point signal for FEAT-09.SPEC-002 opens |
| invoice_list_empty_state_shown | scope | List loads with zero invoices | N/A -- no metric measures the empty state; retained to distinguish "no invoices yet" from a load failure in product telemetry |
| invoice_list_row_selected | invoice reference, scope | User taps an invoice row | supports success-metrics.md: "Invoice Auto-Generation Accuracy" (a row selected and opened is the precondition for Nadia or Owen ever noticing an amount, currency, or tax discrepancy, which this metric measures downstream on FEAT-09.SPEC-002) |

## Acceptance Criteria

**FEAT-09.SPEC-001-AC-01:** Given Nadia opens the invoices area of a project with three invoices, when the screen loads, then all three appear as rows ordered newest-issued first.

**FEAT-09.SPEC-001-AC-02:** Given a project has never had an invoice, when Nadia opens its invoices area, then she sees "No invoices issued" with a "New Invoice" action.

**FEAT-09.SPEC-001-AC-03:** Given Owen opens "Invoices" from his portal home and his company has invoices across two different projects, when the screen loads, then both projects' invoices appear in one list, each row labeled with its project.

**FEAT-09.SPEC-001-AC-04:** Given Priya is signed into her portal, when she looks for an "Invoices" entry, then none is shown anywhere in her navigation.

**FEAT-09.SPEC-001-AC-05:** Given Dana is inside a logged support session, when she opens the invoice list, then she sees every invoice read-only with no "New Invoice" action.

**FEAT-09.SPEC-001-AC-06:** Given Nadia taps an invoice row, when the tap registers, then she lands on FEAT-09.SPEC-002 for that invoice.

**FEAT-09.SPEC-001-AC-07:** Given Nadia taps "New Invoice", when the tap registers, then she lands on FEAT-09.SPEC-003.

**FEAT-09.SPEC-001-AC-08:** Given the list fails to load, when the failure occurs, then Nadia sees "Couldn't load invoices. Check your connection and try again." with a Retry action.

**FEAT-09.SPEC-001-AC-09:** Given Owen loses connectivity while viewing his invoice list, when the loss occurs, then the last-loaded rows remain visible under a "You're offline" banner and refresh silently once connectivity returns.

**FEAT-09.SPEC-001-AC-10:** Given a new invoice is generated automatically while Nadia has the list open, when generation completes, then the new row appears in the list without a manual reload.

**FEAT-09.SPEC-001-AC-11:** Given Owen's session expires while he is on this screen, when he attempts to interact with it, then he sees FEAT-05.SPEC-002's expired-link explanation with a one-tap way to request a fresh link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (empty, loading, populated, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Invoice Detail

## Overview

**Name:** Invoice Detail
**ID:** FEAT-09.SPEC-002
**Type:** Screen
**Purpose:** Shows one invoice's amount, status, due date, reminder history, and pay-link/download controls, with what each side may do differentiated by role.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- Displaying one invoice's full content: amount, currency, tax line, status, issue and due dates, business/billing details, triggering event, and reminder history
- The pay-link status banner (ready / not yet available / temporarily unavailable), sourced from FEAT-09.SPEC-009
- Downloading a printable copy of the invoice, credit note, or receipt
- The entry point into issuing a credit note against this invoice, once it is Sent or later
- Owen's pay-link action, and Nadia's entry point to recording an off-platform payment or a refund (both owned by other features)

**Non-Goals:**
- Generating or sending the invoice -- owned entirely by FEAT-09.SPEC-004 (automatic) and FEAT-09.SPEC-005 (manual); this screen only displays the result
- The actual payment flow behind the pay link -- owned by Invoice Payment Processing (FEAT-10); this screen only surfaces the link and its status
- Recording an off-platform payment or a refund -- owned by FEAT-10 and FEAT-25 respectively; this screen only provides the entry points from the freelancer's side
- Editing a sent invoice's content -- excluded per FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules): once Sent or later, no edit control is ever shown; a correction is always a new credit note (FEAT-09.SPEC-003) or a new invoice

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-001 (Invoice List) | Nadia or Owen taps an invoice row | Invoice reference |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Owen taps the invoice email's pay-link CTA, or Nadia taps her copy confirmation's CTA | Invoice reference |
| FEAT-12 (Freelancer Financial Dashboard) | Nadia opens a specific invoice from a client drill-down | Invoice reference |
| FEAT-31 (Operator Support Access) | Dana opens an invoice inside a logged support session | Invoice reference, read-only session context |
| FEAT-25.SPEC-007 (Refund & Cancellation Notification) | The recipient taps the email's "View invoice" CTA | Invoice reference for the affected invoice |
| FEAT-25.SPEC-008 (Payment Reversal Notification) | Nadia taps the email's "View invoice" CTA | Invoice reference for the disputed invoice |
| FEAT-13.SPEC-001 (Activity Trail) | Nadia taps an invoice-related trail entry or its affected-record link | Invoice reference |
| FEAT-28.SPEC-001 (Global Search) | Nadia selects an Invoice search result | Invoice reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full content of any of her invoices | Download a copy; issue a credit note (via FEAT-09.SPEC-003) once Sent or later; record an off-platform payment (FEAT-10) or a refund (FEAT-25) from this screen's entry points | -- |
| Owen (Client Primary Contact) | Full content of his own company's invoices, own-only | Follow the pay link (when ready); download a copy | -- |
| Priya (Client Reviewer Contact) | None -- per FEAT-09.SPEC-006, invoice content is hidden entirely from Reviewer contacts | None | A direct link to this screen redirects to her portal home with no error message and no partial content ever rendered |
| Dana (Support Operator) | Full content, read-only, inside a logged support session (FEAT-31) | View only | Every action control (download, pay, credit note, off-platform payment, refund) is not rendered in Dana's session; she sees content only |
| Unauthenticated | No | No | Redirected to sign-in (FEAT-05.SPEC-001 for a client contact) |
| Expired session | No | No | Client contact: FEAT-05.SPEC-002's expired-link explanation with a one-tap fresh-link request; Nadia: redirected to sign-in |

## Layout and Content

**Header:** Invoice number and status badge, with a back control returning to the entry point (Invoice List or dashboard).

**Body, in order:**
- **Summary card** (the same visual pattern as FEAT-09.SPEC-001's list row, at full density): triggering event label, amount/currency/total with the tax line broken out, issue date, due date.
- **Business and billing details:** Nadia's business name/address/tax ID and the client's billing name/address, as printed on the invoice.
- **Pay-link status banner:** one of the three states defined by FEAT-09.SPEC-009 (ready to pay, online payment not yet available, temporarily unavailable), with Owen's "Pay now" action shown only in the ready state.
- **Reminder history:** a chronological list of Reminder Log entries (day 3, day 10, manual, pause state), visible to Nadia only.
- **Download control:** available to Nadia and Owen for this invoice, or its credit note/receipt counterpart if one exists.
- **Correction area** (Nadia only, once status is Sent or later): "Correct with a credit note" action, replacing any would-be edit control per the shared "not yet edited after sending" convention.
- **Off-platform actions area** (Nadia only): "Record a payment received elsewhere" (FEAT-10) and, once paid, "Record a refund" (FEAT-25).

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order listed above; the pay-link banner and its action remain visible without scrolling past the summary card.
- **Medium size class and above:** Summary card and business/billing details sit side by side; the remaining sections continue to stack full-width beneath them.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to entry point | Screen closes | Standard transition |
| "Pay now" (Owen, ready state only) | Tap | Navigate to FEAT-10 (pay invoice) | Screen transitions | Standard transition |
| Download control | Tap | Produces a printable copy of the invoice, credit note, or receipt | None -- read-only action | A file is offered for save; a brief "Preparing your copy..." indicator shows while it generates |
| "Correct with a credit note" (Nadia, Sent+ only) | Tap | Navigate to FEAT-09.SPEC-003 with this invoice pre-selected for correction | Screen transitions | Standard transition |
| "Record a payment received elsewhere" (Nadia) | Tap | Navigate to FEAT-10's manual-payment entry | Screen transitions | Standard transition |
| "Record a refund" (Nadia, Paid invoices only) | Tap | Navigate to FEAT-25's refund entry | Screen transitions | Standard transition |
| Reminder history entries | Display only | None | None | Read-only list; no interaction |

### Accessibility Notes

- **Focus order:** Back control -> status badge -> summary card -> business/billing details -> pay-link banner (and its action, if present) -> reminder history -> download control -> correction/off-platform actions.
- **Status announcements:** A live pay-link status change (e.g., from ready to temporarily unavailable, per FEAT-09.SPEC-009) while this screen is open is announced to assistive technology.
- **Keyboard alternatives:** Every action is keyboard-reachable; the download control has no pointer-only gesture equivalent.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton in place of the summary card and sections | Screen first opens | Data finishes loading |
| Empty | N/A -- this screen is never opened without a resolved invoice reference; it has no zero-data state. Every entry point (FEAT-09.SPEC-001, FEAT-09.SPEC-010, FEAT-12, FEAT-31) carries a specific invoice reference as context, so a load always resolves to either Populated or Error, never to an invoice-less screen | Not applicable | Not applicable |
| Populated | Full content as described above | Load completes successfully | Screen closes or the invoice's live status changes (re-renders in place) |
| Download in progress | "Preparing your copy..." indicator on the download control | Download tapped | Copy is ready, or generation fails |
| Error | Error banner "Couldn't load this invoice. Check your connection and try again." with Retry | Initial load fails | Retry succeeds |
| Offline/Degraded | The last-loaded invoice detail remains visible under a "You're offline -- this may be out of date" banner; every action requiring connectivity (pay, download, credit note, off-platform recording) is disabled with a "Reconnect to continue" note | Connectivity is lost while viewing, or the screen is opened while offline | Connectivity returns and the screen re-fetches the current state |

## Validation Rules

Not applicable -- this screen has no user input beyond navigation and action taps; all validation for the actions it launches into lives in the destination specs (FEAT-09.SPEC-003, FEAT-10, FEAT-25).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Pay now" | FEAT-10 (pay invoice) | FEAT-10 |
| "Correct with a credit note" | FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | -- |
| "Record a payment received elsewhere" | FEAT-10 (manual payment entry) | FEAT-10 |
| "Record a refund" | FEAT-25 (refund entry) | FEAT-25 |

## Data Model

**Creates:** None.
**Reads:** Invoice -- `invoice_number`, `project`, `triggering_event`, `amount`, `currency`, `tax_label`, `tax_rate`, `total`, business/billing details, `issue_date`, `due_date`, `status`, `pay_link availability`. Reminder Log -- `reminder_type`, `scheduled_for`, `sent_at`, `pause_state` (Nadia only). Payment Account Connection -- status, read via FEAT-09.SPEC-009 to derive the pay-link banner.
**Updates:** None directly -- this screen never writes to the Invoice; downstream actions (pay, credit note, off-platform payment, refund) write through their own owning specs.
**Deletes:** None.

## Business Rules

- FEAT-09.SPEC-006 governs who reaches this screen and what each role may do; this screen's Access and Visibility table is consistent with it.
- FEAT-09.SPEC-008 governs the "not yet edited after sending" convention: no edit control is ever shown once `status` is Sent or later; only "Correct with a credit note" is offered.
- FEAT-09.SPEC-009 governs the exact wording and behavior of the pay-link status banner; this screen renders that spec's output without deciding it independently.
- The reminder history section reads the Reminder Log (owned by FEAT-11) and is visible to Nadia only, since Owen has no entitlement to see the freelancer's follow-up cadence, per the Access Matrix.

## Edge Cases

- **The invoice's status changes (e.g., to Paid, or to Overdue) while this screen is open** -- No live conflict, since this screen performs no writes of its own: the displayed status re-renders to the new value in place, consistent with the read-only nature of this view; no reload or user action is required.
- **Owen opens this screen for an invoice with no connected payment account** -- The pay-link banner shows the "online payment not yet available" wording per FEAT-09.SPEC-009, and no "Pay now" action is rendered.
- **Nadia taps "Correct with a credit note" on an invoice that another session has already corrected** -- Rejected-with-refresh: the screen reloads to show `status: Corrected` and the existing credit note link, rather than opening a second correction flow against an invoice already corrected.
- **The download control is tapped while offline** -- The action is disabled with the "Reconnect to continue" note from the Offline/Degraded state; no partial or corrupted copy is ever produced.
- **Nadia or Owen attempts a manual issuance action or opens this detail page while offline** -- Per the Brief's Side-Effect Inventory: a clear "reconnect" state is shown; the action never appears to succeed, and the last-known invoice detail is shown as possibly out of date if cached, consistent with the Offline/Degraded state above.
- **Dana opens this screen inside a support session and the invoice is later corrected by Nadia in a separate session** -- Dana's read-only view re-renders to the corrected state on next refresh; she never sees a control to act on either version.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-001 (Invoice List) | Navigation (inbound) | Row selection arrives here |
| FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | Navigation (outbound) | "Correct with a credit note" opens the correction flow |
| FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules) | References (inbound) | Governs the Access and Visibility table |
| FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules) | References (inbound) | Governs the no-edit-after-sending convention |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (inbound) | Governs the pay-link banner's content |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Navigation (inbound) | Email CTAs arrive here |
| FEAT-10 (Invoice Payment Processing) | Navigation (outbound) | "Pay now" and "Record a payment received elsewhere" |
| FEAT-25 (Refund & Cancelled Project Handling) | Navigation (outbound) | "Record a refund" |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support-session entry |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| invoice_detail_viewed | invoice reference, viewer role, invoice status at view time | Screen finishes loading | supports success-metrics.md: "Invoice Auto-Generation Accuracy" (viewing is the moment either party could notice, and report, an incorrect amount, currency, or tax line) |
| invoice_copy_downloaded | invoice reference, document type (invoice / credit note / receipt), viewer role | Download completes successfully | N/A -- no success-metrics.md metric measures copy downloads; retained since product-features.md's Signals field for FEAT-09 names `invoice_copy_downloaded` explicitly as a required signal |
| invoice_correction_entry_opened | invoice reference | Nadia taps "Correct with a credit note" | N/A -- no connected metric measures correction-flow entry; retained as the upstream signal feeding FEAT-09.SPEC-005's correction-recording events |

## Acceptance Criteria

**FEAT-09.SPEC-002-AC-01:** Given Nadia opens an invoice she issued, when the screen loads, then she sees its amount, currency, tax line, status, issue date, due date, and reminder history.

**FEAT-09.SPEC-002-AC-02:** Given Owen opens one of his company's invoices, when the screen loads, then he sees its content but no reminder history section.

**FEAT-09.SPEC-002-AC-03:** Given Priya attempts to open a direct link to an invoice, when the link resolves, then she is redirected to her portal home with no invoice content ever rendered.

**FEAT-09.SPEC-002-AC-04:** Given the invoice's Payment Account Connection is not yet connected, when Owen views the invoice, then the pay-link banner shows "online payment not yet available" wording and no "Pay now" action appears.

**FEAT-09.SPEC-002-AC-05:** Given Owen views an invoice with a ready, connected payment account, when he taps "Pay now", then he navigates to FEAT-10's pay flow.

**FEAT-09.SPEC-002-AC-06:** Given the invoice's status is Sent, when Nadia views the screen, then no edit control is shown anywhere, only "Correct with a credit note."

**FEAT-09.SPEC-002-AC-07:** Given Nadia taps "Correct with a credit note", when the tap registers, then she navigates to FEAT-09.SPEC-003 with this invoice pre-selected for correction.

**FEAT-09.SPEC-002-AC-08:** Given Nadia or Owen taps Download on a Sent invoice, when the copy generates, then a printable file is offered and the invoice_copy_downloaded event is emitted.

**FEAT-09.SPEC-002-AC-09:** Given Dana opens this screen inside a support session, when she looks for any action control, then none is rendered -- only read-only content.

**FEAT-09.SPEC-002-AC-10:** Given the invoice's status changes to Paid while Nadia has this screen open, when the change lands, then the status badge updates in place with no reload required.

**FEAT-09.SPEC-002-AC-11:** Given two of Nadia's sessions both view an invoice already corrected in one of them, when the second session attempts "Correct with a credit note" on the now-stale view, then it is rejected with a refresh showing the existing correction rather than opening a second correction flow.

**FEAT-09.SPEC-002-AC-12:** Given Owen loses connectivity while viewing an invoice, when the loss occurs, then the last-loaded content remains visible under an offline banner and every connectivity-requiring action is disabled with "Reconnect to continue."

**FEAT-09.SPEC-002-AC-13:** Given the initial load fails, when the failure occurs, then Nadia sees "Couldn't load this invoice. Check your connection and try again." with a Retry action.

**FEAT-09.SPEC-002-AC-14:** Given Nadia views a Paid invoice, when she looks for a refund entry point, then "Record a refund" is shown and navigates to FEAT-25.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (loading, empty (N/A), populated, download in progress, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Manual Invoice & Credit Note Issuance

## Overview

**Name:** Manual Invoice & Credit Note Issuance
**ID:** FEAT-09.SPEC-003
**Type:** Screen
**Purpose:** Lets Nadia issue an ad-hoc invoice outside the payment schedule, or a credit note correcting a previously sent invoice, with an adjustable due date.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- The form for a new ad-hoc invoice: description, amount, due date (defaulted, adjustable)
- The form for a credit note against a specific prior invoice, pre-selected when entered from FEAT-09.SPEC-002's correction action
- Submitting either form to FEAT-09.SPEC-005 for validation and recording
- Field-level and submit-time feedback while the submission is processed

**Non-Goals:**
- Validating and recording the submitted invoice or credit note -- owned entirely by FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording); this screen only collects and submits the input
- Automatic invoice generation from a payment-schedule trigger -- owned by FEAT-09.SPEC-004; this screen exists only for issuance outside that schedule
- Choosing which invoice to correct from a general search -- excluded per scope-boundaries.md's Deferral Notes (search is a v1-phase feature, FEAT-28); a credit note is always entered against one specific invoice, either pre-selected from FEAT-09.SPEC-002 or picked from this project's own invoice list (FEAT-09.SPEC-001)
- Recurring or calendar-scheduled billing -- excluded per scope-boundaries.md (SC-14): this screen issues one-off invoices only; an occasional retainer charge is issued here as a single ad-hoc invoice, not a recurring schedule

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-001 (Invoice List) | Nadia taps "New Invoice" | Project reference; form opens in "new ad-hoc invoice" mode with no invoice pre-selected |
| FEAT-09.SPEC-002 (Invoice Detail) | Nadia taps "Issue Credit Note" | The specific invoice reference to correct; form opens in "credit note" mode with that invoice pre-selected and its amount shown for reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Submit an ad-hoc invoice or a credit note | -- |
| Owen (Client Primary Contact) | None -- issuance is a freelancer-only action per the Access Matrix | None | This screen has no entry point in Owen's portal; a direct link redirects him to his portal home |
| Priya (Client Reviewer Contact) | None | None | Same as Owen -- redirected to portal home |
| Dana (Support Operator) | None -- issuing an invoice is excluded from support access per BRIEF.md's "read-only, nothing more" and XBR-29 | None | This screen is never rendered inside a support session; Dana's read-only access stops at FEAT-09.SPEC-001 and FEAT-09.SPEC-002 |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Nadia is redirected to sign-in; entered form data is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title -- "New Invoice" or "Issue Credit Note" depending on mode -- with a back control and a "Submit" action (right-aligned).

**Body -- Ad-hoc invoice mode:**
- Description (text input, required) -- what the charge is for
- Amount (numeric input, required, positive, in the project's set currency)
- Due date (date input, pre-filled from Nadia's `default_payment_terms`, adjustable)

**Body -- Credit note mode:**
- Read-only reference block showing the invoice being corrected: its number, amount, and issue date
- Credit amount (numeric input, required, positive, capped at the original invoice's total)
- Reason (text input, required) -- shown to Owen on the resulting credit note
- Due date -- not applicable to a credit note; this field is not shown in credit-note mode

All fields use one consistent input treatment platform-wide. Required fields are marked with a visual indicator.

**Footer:** None -- Submit is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width; Submit remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to entry point | Screen closes | Confirmation dialog if the form has unsaved input |
| Description / Reason input | Type | Captures text | Field shows entered text | Standard input focus state |
| Amount / Credit amount input | Blur (empty or non-positive) | Triggers validation via FEAT-09.SPEC-007 | Error state on field | "Enter an amount greater than zero." |
| Due date input (ad-hoc only) | Change | Sets the due date, overriding the default-terms pre-fill | Field shows chosen date | Standard input state |
| Submit | Tap | 1. Validate all fields per FEAT-09.SPEC-007. 2. If valid, submit to FEAT-09.SPEC-005 for recording. | Submit shows loading state | Success: navigates to FEAT-09.SPEC-002 for the newly recorded invoice/credit note. Failure: inline error messages, or FEAT-09.SPEC-005's specific rejection reason (e.g., credit amount exceeds the original invoice) |
| Submit (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order (ad-hoc mode):** Back -> Description -> Amount -> Due date -> Submit.
- **Focus order (credit-note mode):** Back -> reference block -> Credit amount -> Reason -> Submit.
- **Validation announcements:** A field entering an error state has its message announced to assistive technology and associated with the field.
- **Keyboard alternatives:** Every action is keyboard-reachable; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton over the due-date field (ad-hoc mode) or over the reference block (credit-note mode), while `default_payment_terms` (ad-hoc) or the original invoice's `invoice_number`/`total`/`issue_date` (credit-note) are fetched; other fields render immediately since they carry no fetched data | Screen first opens | The relevant fetch completes, populating the pre-fill or reference block |
| Empty (default) | Form fields empty except the due-date pre-fill (ad-hoc mode) or the reference block (credit-note mode); Submit enabled | Loading completes | User begins typing or taps Submit |
| Filling | Fields contain user input | User types or changes a field | Submit tapped or user navigates away |
| Submitting | Submit shows a loading indicator, fields disabled | Submit tapped and client-side validation passes | FEAT-09.SPEC-005 returns an outcome |
| Validation Error | Failed fields highlighted with error messages | Client-side validation fails, or FEAT-09.SPEC-005 rejects the submission | User corrects the field(s) |
| Error | Error banner "Couldn't submit this invoice. Check your connection and try again." with Retry, form data preserved | Submission fails for a reason other than validation (e.g., connectivity) | User taps Retry and the resubmission succeeds |
| Offline/Degraded | A "You're offline -- reconnect to submit" banner is shown; the form remains editable but Submit is disabled; the action never appears to succeed while offline | Connectivity is lost while the screen is open, or the screen is opened while offline | Connectivity returns and Submit becomes available again |

## Validation Rules

Validation governed by FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules). See that spec for the amount-positivity rule, the due-date derivation and override rule, and the credit-amount-cannot-exceed-original rule. This screen applies validation on field blur and on submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back control (no unsaved changes) | Entry point (FEAT-09.SPEC-001 or FEAT-09.SPEC-002) | -- |
| Successful submission | FEAT-09.SPEC-002 (Invoice Detail) for the newly recorded invoice or credit note | -- |
| Cancel with unsaved changes | Entry point, after confirmation | -- |

## Data Model

**Creates:** None directly -- this screen submits input to FEAT-09.SPEC-005, which creates the Invoice record (ad-hoc invoice or credit note).
**Reads:** Invoice (credit-note mode only) -- `invoice_number`, `total`, `issue_date` of the invoice being corrected, for the read-only reference block. Freelancer Account -- `default_payment_terms`, to pre-fill the due date.
**Updates:** None directly.
**Deletes:** None.

## Business Rules

- FEAT-09.SPEC-007 governs numbering, required content, amount validation, and due-date derivation -- this screen applies it rather than re-deriving it.
- FEAT-09.SPEC-008 governs correction rules: a credit note is always a new, linked Invoice record; this screen never edits the original invoice's fields.
- Submission is disabled while offline (per the States table), consistent with the Brief's Side-Effect Inventory: "the action never appears to succeed" without connectivity.
- Only Nadia can reach this screen, per the Access Matrix -- issuance is never exposed to any client contact or to Dana.

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Submit twice rapidly** -- Second tap is ignored while the first submission is in progress.
- **Credit amount entered exceeds the original invoice's total** -- Blocked at submit time by FEAT-09.SPEC-007 with "A credit note cannot exceed the original invoice's total." Field-level error shown; submission does not proceed.
- **The invoice being corrected is corrected by another session between this screen opening and Submit** -- Rejected-with-refresh, consistent with the Invoice entity's Contention note: the submission is refused with "This invoice was already corrected. View the existing credit note." and the reference block updates to the current state.
- **Network failure during submission** -- Error banner: "Couldn't submit this invoice. Check your connection and try again." with Retry; entered data is preserved.
- **Nadia opens this screen offline** -- The form is viewable and editable, but Submit is disabled with the offline banner until connectivity returns, per the Brief's Side-Effect Inventory offline behavior for this screen.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-001 (Invoice List) | Navigation (inbound) | "New Invoice" arrives here in ad-hoc mode |
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (inbound) | "Issue Credit Note" arrives here in credit-note mode |
| FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording) | Triggers (outbound) | Submission hands off to this automation for validation and recording |
| FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules) | References (inbound) | Validation rules applied to form fields |
| FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules) | References (inbound) | Governs why a credit note is a new record, never an edit |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| manual_invoice_form_opened | mode (ad_hoc / credit_note) | Screen opens | N/A -- no success-metrics.md metric tracks form opens directly; retained as the funnel entry for the manual_invoice_issued signal below |
| manual_invoice_submitted | mode, outcome (accepted / rejected) | Submit completes | supports success-metrics.md: "Invoice Auto-Generation Accuracy" (a rejected ad-hoc submission -- e.g., a credit note exceeding the original total -- is the manual-path counterpart to the accuracy this metric measures on the automatic path) |
| manual_invoice_form_abandoned | mode, fields filled | User discards unsaved changes | N/A -- no connected metric measures abandonment; retained for product visibility into a form that goes unfinished |

## Acceptance Criteria

**FEAT-09.SPEC-003-AC-01:** Given Nadia taps "New Invoice" from a project's invoice list, when the form opens, then it is in ad-hoc mode with an empty description and amount, and a due date pre-filled from her default payment terms.

**FEAT-09.SPEC-003-AC-02:** Given Nadia taps "Issue Credit Note" from an invoice's detail screen, when the form opens, then it is in credit-note mode with that invoice's number, total, and issue date shown read-only.

**FEAT-09.SPEC-003-AC-03:** Given Nadia enters a description and a positive amount and taps Submit in ad-hoc mode, when submission succeeds, then she lands on FEAT-09.SPEC-002 for the newly recorded invoice.

**FEAT-09.SPEC-003-AC-04:** Given Nadia leaves the amount field empty and taps Submit, when validation runs, then the amount field shows "Enter an amount greater than zero." and submission does not proceed.

**FEAT-09.SPEC-003-AC-05:** Given Nadia enters a credit amount greater than the original invoice's total, when she taps Submit, then she sees "A credit note cannot exceed the original invoice's total." and submission does not proceed.

**FEAT-09.SPEC-003-AC-06:** Given Nadia adjusts the pre-filled due date on an ad-hoc invoice, when she submits, then the invoice is recorded with her chosen due date, not the default.

**FEAT-09.SPEC-003-AC-07:** Given Nadia navigates away with unsaved input, when she taps the back control, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-09.SPEC-003-AC-08:** Given Nadia taps Submit twice in rapid succession, when the first submission is still in flight, then the second tap has no effect.

**FEAT-09.SPEC-003-AC-09:** Given the invoice Nadia is correcting was already corrected in another session, when she taps Submit, then she sees "This invoice was already corrected. View the existing credit note." and no second credit note is recorded.

**FEAT-09.SPEC-003-AC-10:** Given a network failure occurs during submission, when the failure is detected, then Nadia sees "Couldn't submit this invoice. Check your connection and try again." with her entered data preserved.

**FEAT-09.SPEC-003-AC-11:** Given Nadia opens this screen while offline, when she attempts to submit, then Submit is disabled and a "reconnect to submit" banner is shown; the action never appears to succeed.

**FEAT-09.SPEC-003-AC-12:** Given Owen attempts to reach this screen directly, when the link resolves, then he is redirected to his portal home with no form ever rendered.

**FEAT-09.SPEC-003-AC-13:** Given Dana is inside a logged support session, when she looks for an issuance entry point, then none exists anywhere in her session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 7 (loading, empty, filling, submitting, validation error, error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Automatic Invoice Generation

## Overview

**Name:** Automatic Invoice Generation
**ID:** FEAT-09.SPEC-004
**Type:** Automation
**Purpose:** On a deposit acceptance, milestone approval, or project completion, generates the correct invoice with no freelancer action and hands it off to be sent.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- Receiving the deposit, next-invoice, and on-completion hand-offs from FEAT-03, FEAT-08, and FEAT-01 respectively
- Applying FEAT-09.SPEC-007's numbering, content, and due-date rules to produce a complete Invoice record
- Checking pay-link availability via FEAT-09.SPEC-009 and carrying its wording into the record
- Setting the Invoice to `Sent` and handing off to FEAT-09.SPEC-010 for the send notification
- Blocking generation, with a surfaced warning rather than silent failure, when the freelancer's or client's required billing details are missing

**Non-Goals:**
- Deciding whether a deposit, approval, or completion event itself occurred -- owned entirely by FEAT-03, FEAT-08, and FEAT-01, which fire this automation only after their own triggers are confirmed
- Ad-hoc invoicing or credit notes -- owned by FEAT-09.SPEC-005, a separate automation for issuance outside this schedule-driven path
- The actual email send and delivery/bounce status -- owned by the Transactional Email Delivery capability via FEAT-09.SPEC-010; this automation only hands off a ready invoice
- Currency conversion or tax calculation -- excluded per scope-boundaries.md (SC-16): this automation applies the project's already-configured currency and tax line (FEAT-15) rather than computing either

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Deposit due on acceptance | FEAT-03.SPEC-003 (Acceptance Recording) | Fires when the Payment Schedule, as it stood at acceptance, includes a deposit (XBR-01) | Project reference, Payment Schedule's deposit terms, accepted-at timestamp |
| Next invoice on milestone approval | FEAT-08.SPEC-004 (Next-Invoice Trigger) | Fires when the approved milestone's `payment_trigger` marks it as invoice-on-approval, per the schedule as it stood at approval (XBR-02) | Milestone reference, its price, Project reference, approval timestamp |
| On-completion invoice | FEAT-01.SPEC-006 (Completion Invoice Trigger) | Fires when the Payment Schedule's structure includes a `completion_amount`, at the moment a project is marked complete (XBR-03) | Project reference, Payment Schedule's completion_amount, completed_at timestamp |

## Processing Logic

1. Receive the triggering event (deposit, next-invoice, or on-completion) and its Project, amount source, and timestamp from the source spec.
2. Read the Project's currency and tax line (FEAT-15), the Client's billing details, and the Freelancer Account's business details and `default_payment_terms`.
3. Check that both the Freelancer Account's business details and the Client's billing details are present (XBR-16), and that the Project's currency (and tax line, if configured) is set (XBR-17). If the freelancer's business details are missing, stop and produce the "generation blocked -- missing freelancer business details" outcome. If the client's billing details are missing, stop and produce the "generation blocked -- missing client billing details" outcome. If the currency/tax configuration is missing, stop and produce the "generation blocked -- missing currency/tax configuration" outcome. Each reason is reported with its own exact message per FEAT-09.SPEC-007 -- never a single generic message.
4. Apply FEAT-09.SPEC-007's rules: assign the next sequential `invoice_number` for this freelancer, set `amount`, `currency`, `tax_label`, `tax_rate`, and `total` to match the triggering event's price plus tax exactly, set `issue_date` to now, and derive `due_date` from `default_payment_terms`.
5. Check pay-link availability via FEAT-09.SPEC-009 using the current Payment Account Connection status, and record the resulting wording state on the Invoice.
6. Create the Invoice record with `status: Generated`, `triggering_event` set to the source (deposit / milestone approval / completion), and a reference back to the Project and the triggering record (accepted proposal, approved milestone, or completed project).
7. Immediately transition the new Invoice to `status: Sent` and hand off to FEAT-09.SPEC-010 to send it to Owen with a confirmation copy to Nadia.
8. Return the outcome to the triggering spec for observability; this automation does not itself confirm email delivery -- that confirmation belongs to FEAT-09.SPEC-010.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Invoice generated and sent | Billing details are complete; generation and the send hand-off both succeed | New Invoice created with `status: Sent`, numbered, dated, and priced per FEAT-09.SPEC-007 | Owen and Nadia see the invoice arrive via FEAT-09.SPEC-010; the invoice appears in FEAT-09.SPEC-001 and FEAT-09.SPEC-002 with no visible delay from the triggering event | FEAT-09.SPEC-007, FEAT-09.SPEC-009, FEAT-09.SPEC-010, FEAT-09.SPEC-001, FEAT-09.SPEC-002 |
| Generation blocked -- missing freelancer business details | The Freelancer Account's business details are incomplete at the moment of firing | No Invoice created | Nadia sees a non-blocking warning on the Project, using FEAT-09.SPEC-007's exact freelancer-side message: "Your business details aren't complete yet. Add them in Settings before this invoice can be sent." with a link to the missing details' settings screen | FEAT-01.SPEC-005 (project-level warning), FEAT-21 (business details) |
| Generation blocked -- missing client billing details | The Client's billing details are incomplete at the moment of firing | No Invoice created | Nadia sees a non-blocking warning on the Project, using FEAT-09.SPEC-007's exact client-side message: "This client's billing details are incomplete. Add a billing name and address before sending an invoice." with a link to the missing details' settings screen | FEAT-01.SPEC-005 (project-level warning), FEAT-01 (client billing details) |
| Generation blocked -- missing currency/tax configuration | The Project's currency (and tax line) was never configured before the trigger fires | No Invoice created | Nadia sees a non-blocking warning on the Project, using FEAT-09.SPEC-007's exact message: "This project's currency isn't set yet. Set it before the first invoice." with a link to the currency/tax settings screen (FEAT-15) | FEAT-01.SPEC-005 (project-level warning), FEAT-15 (currency & tax configuration) |
| Send hand-off failure (invoice created, notification cannot be reached) | Invoice creation and numbering succeed, but FEAT-09.SPEC-010 cannot be reached at hand-off | Invoice remains created with `status: Sent` already set -- the send is retried, not the creation | The invoice appears immediately in Nadia's and Owen's lists and details even before the email arrives; the send is retried automatically without creating a duplicate invoice record, per the Brief's Side-Effect Inventory | FEAT-09.SPEC-010 (retried send) |
| No trigger applicable (defensive no-action) | The triggering spec fires this automation but its own conditions (deposit/next-invoice/completion price) resolve to zero or undefined | No Invoice created | Nothing is shown -- this path is a defensive guard; the triggering specs (FEAT-03.SPEC-003, FEAT-08.SPEC-004, FEAT-01.SPEC-006) already filter out non-invoicing cases before calling this automation, so this outcome is not expected to occur in normal operation | None |

## Data Model

**Reads:** Payment Schedule -- `structure`, `deposit_amount`, `completion_amount`, as they stood at the triggering moment. Milestone -- `price`, `payment_trigger`. Project -- currency and tax configuration (FEAT-15), reference. Client -- billing details. Freelancer Account -- business details, `default_payment_terms`. Payment Account Connection -- status, via FEAT-09.SPEC-009.
**Creates:** Invoice -- `invoice_number`, `project`, `triggering_event`, `amount`, `currency`, `tax_label`, `tax_rate`, `total`, business/billing details, `issue_date`, `due_date`, `status`, `pay_link availability`.
**Updates:** Invoice -- `status` (Generated to Sent, within this single automation run).
**Deletes:** None.

## Business Rules

- XBR-01, XBR-02, XBR-03: this automation is the single owner of invoice creation for all three automatic triggers; the source specs own firing the trigger, not the invoice itself.
- XBR-16: sending is blocked until both the freelancer's business details and the client's billing details exist -- enforced here as the "generation blocked" outcome, never a silent skip.
- XBR-17: currency and tax line are applied exactly as configured by FEAT-15 at the moment of generation; this automation computes no tax of its own.
- Amount must match the triggering milestone/deposit/completion price plus tax exactly (FEAT-09.SPEC-007) -- there is no rounding, discounting, or adjustment step in this automation.
- A failed send is retried without creating a duplicate invoice record, per the Brief's Side-Effect Inventory -- the Invoice's creation and its send are treated as separate steps for retry purposes, so a retried send never re-runs invoice creation.

## Edge Cases

- **The Project's currency or tax line was never configured before the first trigger fires** -- Generation is blocked with the dedicated "missing currency/tax configuration" outcome (XBR-17: currency and tax must be set before the first invoice); Nadia sees FEAT-09.SPEC-007's exact message: "This project's currency isn't set yet. Set it before the first invoice."
- **Nadia is offline when a trigger fires** -- Generation still proceeds automatically in the background; there is no freelancer action required and none is blocked by her connectivity, per the Brief's Side-Effect Inventory (Offline/Degraded: N/A for this automation).
- **The Payment Account Connection has no connected account at generation time** -- The invoice still generates and sends; FEAT-09.SPEC-009 supplies the "online payment not yet available" wording rather than blocking the send.
- **Two different triggers for the same project fire at effectively the same time (e.g., a milestone approval and a project completion within the same instant)** -- Concurrent trigger firing: each trigger's source spec calls this automation independently with its own amount and triggering-event reference; this automation assigns each a separate, sequentially numbered Invoice, since invoice numbering is a single, freelancer-wide sequence that serializes concurrent assignments rather than colliding on one number.
- **This automation is invoked for a new trigger while a previous invocation for a different trigger on the same project is still in flight** -- Each run is independent and reads only the data relevant to its own trigger; the only shared resource is the sequential `invoice_number` counter, which FEAT-09.SPEC-007 guarantees assigns uniquely even under concurrent runs.
- **The billing-details check passes, but the Client's billing details are cleared by Nadia between the trigger firing and this automation's read** -- Extremely narrow window; if the read in step 3 finds the details missing, the outcome is "generation blocked," consistent with the check being evaluated at read time, not at trigger time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-003 (Acceptance Recording) | Triggered by (inbound) | Fires the deposit-invoice trigger on acceptance |
| FEAT-08.SPEC-004 (Next-Invoice Trigger) | Triggered by (inbound) | Fires the next-invoice trigger on approval |
| FEAT-01.SPEC-006 (Completion Invoice Trigger) | Triggered by (inbound) | Fires the on-completion trigger |
| FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules) | References (inbound) | Numbering, content, and due-date rules applied at creation |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (inbound) | Determines the pay-link wording carried onto the new invoice |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Triggers (outbound) | Sends the resulting invoice to Owen and its copy to Nadia |
| FEAT-01.SPEC-005 (Project Detail) | Affects (outbound) | Shows the non-blocking warning when generation is blocked |
| FEAT-15 (Currency & Tax Handling) | References (inbound) | Supplies the currency and tax configuration applied |
| FEAT-21 (Settings & Account Management) | References (inbound) | Supplies the freelancer's business details and default payment terms |

## Analytics and Success Signals

- **invoice_generated** (triggering event type, invoice reference) -- supports success-metrics.md: "Invoice Auto-Generation Accuracy"
- **invoice_sent** (invoice reference, triggering event type) -- N/A -- no success-metrics.md metric is connected to the send moment itself distinct from generation accuracy; retained per product-features.md's Signals field, which names `invoice_sent` explicitly for this feature
- **invoice_generation_blocked** (reason: missing_freelancer_details / missing_client_details / missing_currency_tax_config) -- N/A -- no Stage 2 metric measures blocked generations directly; retained so a billing-completeness gap is observable rather than silently absorbed, since "Invoice Auto-Generation Accuracy" measures the correctness of invoices that were generated, not ones that were blocked
- **invoice_send_handoff_retried** (retry_count) -- N/A -- no connected metric covers infrastructure hand-off retries; retained as this automation's only signal of that failure mode, mirroring the pattern used by FEAT-08.SPEC-004's `milestone_invoice_handoff_failed`

## Acceptance Criteria

**FEAT-09.SPEC-004-AC-01:** Given a proposal is accepted and the Payment Schedule includes a deposit, when FEAT-03.SPEC-003 fires this automation, then a deposit invoice is generated with the correct amount, numbered sequentially, and immediately sent to Owen.

**FEAT-09.SPEC-004-AC-02:** Given a milestone is approved and its `payment_trigger` marks it to invoice on approval, when FEAT-08.SPEC-004 fires this automation, then the next invoice is generated for that milestone's price and sent.

**FEAT-09.SPEC-004-AC-03:** Given a project is marked complete and the schedule includes a `completion_amount`, when FEAT-01.SPEC-006 fires this automation, then the on-completion invoice is generated and sent.

**FEAT-09.SPEC-004-AC-04:** Given the Freelancer Account's business details are incomplete at the moment of firing, when this automation runs, then no invoice is created and Nadia sees the non-blocking warning "Your business details aren't complete yet. Add them in Settings before this invoice can be sent."

**FEAT-09.SPEC-004-AC-05:** Given the Client's billing details are incomplete at the moment of firing, when this automation runs, then no invoice is created and Nadia sees the non-blocking warning "This client's billing details are incomplete. Add a billing name and address before sending an invoice."

**FEAT-09.SPEC-004-AC-06:** Given Nadia has no connected Payment Account Connection at generation time, when the invoice is generated, then it still issues and sends, carrying the "online payment not yet available" wording from FEAT-09.SPEC-009.

**FEAT-09.SPEC-004-AC-07:** Given the invoice is created successfully but the send hand-off to FEAT-09.SPEC-010 fails, when the failure occurs, then the invoice remains visible in Nadia's and Owen's lists, and the send is retried automatically without creating a second invoice record.

**FEAT-09.SPEC-004-AC-08:** Given a milestone approval and a project completion fire at effectively the same time for the same project, when both invocations run, then each produces its own separately numbered invoice with no collision.

**FEAT-09.SPEC-004-AC-09:** Given Nadia is offline when a trigger fires, when the trigger fires, then the invoice still generates and sends with no freelancer action required.

**FEAT-09.SPEC-004-AC-10:** Given the project's currency and tax line were never configured, when a trigger fires, then generation is blocked with the warning "This project's currency isn't set yet. Set it before the first invoice."

**FEAT-09.SPEC-004-AC-11:** Given the deposit invoice generates successfully, when the invoice_generated event is emitted, then it carries the triggering event type and the invoice reference.

**FEAT-09.SPEC-004-AC-12:** Given generation is blocked for missing billing details, when the invoice_generation_blocked event is emitted, then it carries the specific reason (missing_freelancer_details / missing_client_details / missing_currency_tax_config).

**FEAT-09.SPEC-004-AC-13:** Given this automation runs for a second trigger while a first trigger's run on the same project is still completing its send hand-off, when both runs execute, then neither is blocked by the other, since each reads only its own triggering data and shares no state beyond the sequential invoice-numbering counter.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (deposit, next-invoice, completion) | 3 |
| Outcome Paths | 6 (generated & sent, blocked -- missing freelancer details, blocked -- missing client details, blocked -- missing currency/tax, hand-off failure, no-action) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Manual Invoice & Credit Note Recording

## Overview

**Name:** Manual Invoice & Credit Note Recording
**ID:** FEAT-09.SPEC-005
**Type:** Automation
**Purpose:** Validates and writes an ad-hoc invoice or a credit note submitted from FEAT-09.SPEC-003, marking the original invoice Corrected when a credit note supersedes it.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- Validating and recording an ad-hoc invoice submitted from FEAT-09.SPEC-003
- Validating and recording a credit note against a specific prior invoice, applying the immutability and correction rules
- Marking the original invoice's `status: Corrected` at the moment its credit note is recorded
- Handing off the newly recorded invoice or credit note to FEAT-09.SPEC-010 for sending

**Non-Goals:**
- Collecting the form input itself -- owned entirely by FEAT-09.SPEC-003; this automation begins at submission
- Automatic invoicing from the payment schedule -- owned by FEAT-09.SPEC-004, a separate automation with its own triggers
- Deciding whether Nadia is authorized to issue invoices at all -- owned by FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules); this automation assumes the submitting user already passed that check on FEAT-09.SPEC-003
- Refund or reversal recording -- excluded per scope-boundaries.md (SC-18): a credit note corrects the invoice record itself, but the actual funds movement behind a refund is owned by Refund & Cancelled Project Handling (FEAT-25)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Ad-hoc invoice submitted | FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance, ad-hoc mode) | Fires when Nadia submits the ad-hoc form after client-side validation passes | Project reference, description, amount, chosen due date |
| Credit note submitted | FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance, credit-note mode) | Fires when Nadia submits the credit-note form after client-side validation passes | Original invoice reference, credit amount, reason |

## Processing Logic

1. Receive the submission (ad-hoc invoice or credit note) and its data from FEAT-09.SPEC-003.
2. Read the Freelancer Account's business details and the Client's billing details; if the freelancer's business details are incomplete, stop and produce the "recording blocked -- missing freelancer business details" outcome; if the client's billing details are incomplete, stop and produce the "recording blocked -- missing client billing details" outcome (same gate as FEAT-09.SPEC-004, applied here for the manual path, with the same side-specific exact messages from FEAT-09.SPEC-007).
3. **Ad-hoc path:** apply FEAT-09.SPEC-007's rules to assign the next sequential `invoice_number`, set `issue_date` to now, set `due_date` to the value Nadia chose (already validated against the default-terms derivation on FEAT-09.SPEC-003), and set `amount`/`currency`/`tax_label`/`tax_rate`/`total` from the entered amount plus the project's configured tax line.
4. **Credit-note path:** re-check at write time that the original invoice's current `status` is not already `Corrected` (guards against a race with another correction). If it is already Corrected, stop and produce the "already corrected" outcome. Otherwise, validate the credit amount does not exceed the original invoice's `total` (FEAT-09.SPEC-007); if it does, stop and produce the "credit amount exceeds original" outcome.
5. **Credit-note path (continued):** create a new Invoice record with `triggering_event: correction`, a reference back to the original invoice, `amount`/`currency` matching the credit amount and the original's currency, `total` set to the negative of the credit amount (a credit rather than a charge), and the entered reason carried as visible content. Assign it the next sequential `invoice_number` per FEAT-09.SPEC-007 -- a credit note is its own numbered record, never a silent adjustment to the original.
6. **Credit-note path (continued):** set the original invoice's `status` to `Corrected` in the same atomic step that creates the credit note, per FEAT-09.SPEC-008.
7. Set the newly recorded invoice or credit note's `status` to `Sent` and hand off to FEAT-09.SPEC-010 for sending to Owen with a confirmation copy to Nadia.
8. Return the outcome to FEAT-09.SPEC-003, which navigates to FEAT-09.SPEC-002 for the newly recorded record on success, or displays the specific rejection reason on failure.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Ad-hoc invoice recorded and sent | Billing details complete; ad-hoc validation passes | New Invoice created (`status: Sent`), numbered and dated per FEAT-09.SPEC-007 | FEAT-09.SPEC-003 navigates Nadia to FEAT-09.SPEC-002 for the new invoice; Owen and Nadia receive FEAT-09.SPEC-010's notification | FEAT-09.SPEC-003, FEAT-09.SPEC-002, FEAT-09.SPEC-010 |
| Credit note recorded, original corrected | Billing details complete; original invoice not already Corrected; credit amount within the original's total | New credit-note Invoice created (`status: Sent`); original invoice's `status` set to `Corrected` | FEAT-09.SPEC-003 navigates Nadia to FEAT-09.SPEC-002 for the credit note; the original invoice's detail now shows `Corrected` with a link to the credit note; Owen and Nadia receive FEAT-09.SPEC-010's notification | FEAT-09.SPEC-003, FEAT-09.SPEC-002, FEAT-09.SPEC-010 |
| Recording blocked -- missing freelancer business details | Freelancer business details incomplete at write time | No record created or updated | FEAT-09.SPEC-003 shows FEAT-09.SPEC-007's exact freelancer-side message: "Your business details aren't complete yet. Add them in Settings before this invoice can be sent." with a link to the relevant settings screen -- rendered identically to FEAT-09.SPEC-004's automatic-path equivalent | FEAT-09.SPEC-003 |
| Recording blocked -- missing client billing details | Client billing details incomplete at write time | No record created or updated | FEAT-09.SPEC-003 shows FEAT-09.SPEC-007's exact client-side message: "This client's billing details are incomplete. Add a billing name and address before sending an invoice." with a link to the relevant settings screen -- rendered identically to FEAT-09.SPEC-004's automatic-path equivalent | FEAT-09.SPEC-003 |
| Already corrected (race) | The original invoice's `status` is already `Corrected` by the time this write is attempted | No record created or updated | FEAT-09.SPEC-003 shows "This invoice was already corrected. View the existing credit note." and links to it | FEAT-09.SPEC-003, FEAT-09.SPEC-002 |
| Credit amount exceeds original | Credit amount is greater than the original invoice's `total` | No record created or updated | FEAT-09.SPEC-003 shows "A credit note cannot exceed the original invoice's total." | FEAT-09.SPEC-003 |
| Recording failure (connectivity or processing error) | The write itself does not complete | No partial record left in either path | FEAT-09.SPEC-003 shows a retry option with entered data preserved | FEAT-09.SPEC-003 |

## Data Model

**Reads:** Invoice (credit-note path) -- `status`, `total`, `currency` of the original. Freelancer Account -- business details. Client -- billing details.
**Creates:** Invoice -- ad-hoc invoice or credit note, per FEAT-09.SPEC-007's numbering and content rules.
**Updates:** Invoice (credit-note path only) -- the original's `status` set to `Corrected`, written exactly once and never altered afterward.
**Deletes:** None.

## Business Rules

- FEAT-09.SPEC-007 governs numbering, required content, amount validation, and due-date derivation for both paths -- applied here, not re-derived.
- FEAT-09.SPEC-008: a sent invoice is never silently edited; a credit note is always a new, linked Invoice record, and marking the original `Corrected` never alters any of the original's other fields (XBR-04).
- The original-invoice re-check (step 4) is authoritative over the screen-time state FEAT-09.SPEC-003 displayed -- consistent with the pattern used elsewhere in this pipeline (e.g., FEAT-03.SPEC-003's write-time re-check) for guaranteeing exactly-once correction.
- Both paths apply the same billing-completeness gate as the automatic path (FEAT-09.SPEC-004), since a manually issued invoice carries the same compliance requirements as an automatically generated one (XBR-16).

## Edge Cases

- **Two credit notes are submitted against the same invoice from two open sessions at effectively the same time** -- Concurrent trigger firing: both invocations reach step 4 near-simultaneously, but only one can win the atomic update in step 6. The first to complete finds the original not yet Corrected and proceeds; the second's re-check then finds `status: Corrected` and returns the "already corrected" outcome. Neither run blocks the other; there is no queuing.
- **Nadia submits a credit note, then submits a second one against the same invoice before the first finishes processing** -- Trigger fires while a previous run is in flight: FEAT-09.SPEC-003 disables Submit while a submission is processing (screen-level debounce), so a second automation run for the same original invoice from the same session cannot start until the first resolves; if it did reach this automation regardless, the same atomic-update guarantee as the concurrent case applies.
- **An ad-hoc invoice is submitted for a project whose currency was never configured** -- Blocked with the same "missing currency/tax configuration" outcome FEAT-09.SPEC-004 uses on the automatic path (XBR-17); Nadia sees FEAT-09.SPEC-007's exact message: "This project's currency isn't set yet. Set it before the first invoice."
- **The credit amount exactly equals the original invoice's total** -- Passes validation (the boundary is inclusive): a full-value credit note is accepted and the original is marked Corrected.
- **The send hand-off to FEAT-09.SPEC-010 fails after recording succeeds** -- The Invoice record (ad-hoc invoice or credit note, and the original's Corrected status) is not rolled back, since it is already evidentiary per XBR-04; the send is retried by FEAT-09.SPEC-010's own retry handling without creating a duplicate record.
- **Nadia submits a credit note against an invoice that has since been paid** -- Recording still proceeds and the original is marked Corrected alongside its existing Paid history; reconciling the payment against the correction (e.g., issuing an actual refund) is a separate action Nadia takes through FEAT-25 from the invoice detail screen, outside this automation's scope.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | Triggered by (inbound) | Both submission paths fire this automation |
| FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules) | References (inbound) | Numbering, content, and due-date rules applied at write time |
| FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules) | References (inbound) | Governs the credit-note-as-new-record and original-marked-Corrected behavior |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Triggers (outbound) | Sends the newly recorded invoice or credit note |
| FEAT-09.SPEC-002 (Invoice Detail) | Affects (outbound) | Displays the recorded result, and the original's Corrected status with a link to its credit note |
| FEAT-25 (Refund & Cancelled Project Handling) | References (outbound) | A credit note against a paid invoice may lead Nadia to a separate refund action there |

## Analytics and Success Signals

- **invoice_manually_issued** (project reference, amount) -- N/A -- no success-metrics.md metric tracks ad-hoc issuance directly; retained per product-features.md's Signals field, which names `invoice_manually_issued` explicitly for this feature
- **invoice_correction_issued** (original invoice reference, credit amount) -- N/A -- no success-metrics.md metric tracks correction volume; retained per product-features.md's Signals field, which names `invoice_correction_issued` explicitly, and because FEAT-09.SPEC-008's immutability guarantee is only verifiable in practice if corrections are observable
- **manual_recording_blocked** (path: ad_hoc / credit_note; reason: missing_billing_details / already_corrected / amount_exceeds_original) -- N/A -- no connected Stage 2 metric measures blocked manual submissions; retained so a rejected manual path is distinguishable from a successful one in product telemetry, mirroring FEAT-09.SPEC-004's `invoice_generation_blocked`

## Acceptance Criteria

**FEAT-09.SPEC-005-AC-01:** Given Nadia submits a valid ad-hoc invoice with billing details complete, when this automation runs, then a new invoice is created, numbered sequentially, and sent to Owen.

**FEAT-09.SPEC-005-AC-02:** Given Nadia submits a credit note against a Sent invoice not already Corrected, when this automation runs, then a new credit-note invoice is created and the original invoice's status is set to Corrected.

**FEAT-09.SPEC-005-AC-03:** Given the Freelancer Account's business details are incomplete, when Nadia submits an ad-hoc invoice, then recording is blocked with "Your business details aren't complete yet. Add them in Settings before this invoice can be sent."

**FEAT-09.SPEC-005-AC-04:** Given the credit amount exceeds the original invoice's total, when Nadia submits the credit note, then it is rejected with "A credit note cannot exceed the original invoice's total."

**FEAT-09.SPEC-005-AC-05:** Given the credit amount exactly equals the original invoice's total, when Nadia submits it, then it passes validation and the original is marked Corrected.

**FEAT-09.SPEC-005-AC-06:** Given the original invoice was already corrected by another session before this write runs, when this automation's re-check executes, then it returns "already corrected" and no second credit note is created.

**FEAT-09.SPEC-005-AC-07:** Given two credit notes against the same invoice are submitted at effectively the same time, when both invocations reach the write-time re-check, then exactly one succeeds and the other returns "already corrected."

**FEAT-09.SPEC-005-AC-08:** Given recording succeeds, when the invoice or credit note is ready, then this automation hands off to FEAT-09.SPEC-010 for sending.

**FEAT-09.SPEC-005-AC-09:** Given the send hand-off fails after recording succeeds, when the failure occurs, then the recorded invoice or credit note is unaffected and the send is retried without duplicating the record.

**FEAT-09.SPEC-005-AC-10:** Given Nadia submits a credit note against an invoice that has already been paid, when recording succeeds, then the original is marked Corrected alongside its existing Paid history, with no automatic refund action taken.

**FEAT-09.SPEC-005-AC-11:** Given the ad-hoc invoice is recorded successfully, when the invoice_manually_issued event is emitted, then it carries the project reference and amount.

**FEAT-09.SPEC-005-AC-12:** Given a manual submission is blocked for any reason, when the manual_recording_blocked event is emitted, then it carries the specific path and reason.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (ad-hoc, credit note) | 2 |
| Outcome Paths | 7 (ad-hoc recorded, credit note recorded, blocked -- missing freelancer details, blocked -- missing client details, already corrected, credit exceeds original, recording failure) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Invoice Access & Role Authorization Rules

## Overview

**Name:** Invoice Access & Role Authorization Rules
**ID:** FEAT-09.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs who may view, issue, follow the pay link on, or download an invoice, including Priya's total exclusion and Dana's read-only support-session visibility.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending
**Governed Entity:** Invoice (access and action authorization, not content)

## Scope and Non-Goals

**In Scope:**
- Authorization rules for every action defined on the Invoice entity by this feature: view (list and detail), issue (ad-hoc/credit note), follow the pay link, download a copy
- The exact denied experience for each role on each action, including Priya's total exclusion from invoice content
- Dana's read-only, session-scoped visibility per Operator Support Access (FEAT-31)

**Non-Goals:**
- Field-level content and validation rules (numbering, amount, due date) -- owned by FEAT-09.SPEC-007
- Immutability and correction behavior -- owned by FEAT-09.SPEC-008
- Pay-link wording and availability -- owned by FEAT-09.SPEC-009; this spec governs only whether a role may follow the link at all, not what it says
- Actions on Payment or Reminder Log records -- owned respectively by FEAT-10 and FEAT-11; this spec governs the Invoice entity only

## Governed Entity

**Entity:** Invoice
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| invoice_number | text (derived) | Unique sequential number per freelancer -- not directly access-relevant, addressed by FEAT-09.SPEC-007 |
| project, triggering_event | reference / enum | Owning project and generation source -- not directly access-relevant |
| amount, currency, tax_label, tax_rate, total | number / text | Financial content -- access-relevant: entirely hidden from Priya |
| business/billing details | text | Access-relevant: entirely hidden from Priya |
| issue_date, due_date | date | Access-relevant: entirely hidden from Priya |
| status | enum | Access-relevant: visibility of status is gated the same as the rest of the invoice's content |
| pay_link availability | derived | Access-relevant: Owen alone may act on it |
| Reminder Log `pause_state` (read-only here; not an Invoice field) | enum (Active, Paused by freelancer, Paused while bank transfer pending) | Access-relevant: the invoice's reminder pause is read from the Reminder Log entity's `pause_state`, which is written only by FEAT-11.SPEC-003 (Nadia's pause/resume) and FEAT-11.SPEC-005 (automatic bank-transfer-pending pause/resume); FEAT-09 never writes it. Visible to Nadia only, not to Owen or Priya |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-001 | Invoice List | Screen entry (navigation hidden/redirected) and per-row rendering |
| FEAT-09.SPEC-002 | Invoice Detail | Screen entry and per-action rendering |
| FEAT-09.SPEC-003 | Manual Invoice & Credit Note Issuance | Screen entry (Nadia-only) |
| FEAT-09.SPEC-004, FEAT-09.SPEC-005 | Automatic / Manual Generation and Recording | Recipient entitlement check before any notification is composed |
| FEAT-11.SPEC-003 | Invoice Reminder Panel | Owns the Nadia-only display and write of the Reminder Log `pause_state` shown on FEAT-09.SPEC-002; FEAT-11.SPEC-005 writes it automatically |
| FEAT-31 | Operator Support Access | Session-scoped read-only rendering for Dana |

## Field Validation Rules

Not applicable to this spec -- field-level validation is owned by FEAT-09.SPEC-007. Every field above is addressed only for its access relevance, per the Governed Entity table.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Content-or-nothing for Priya | All Invoice fields | If the viewing role is Priya, no field of any Invoice is ever rendered, individually or in aggregate (no partial summary, no status-only view) | Not applicable -- no error is shown; the entry point simply does not exist for Priya (see Authorization Rules) |
| Session-scoped visibility for Dana | All Invoice fields | Every field is readable by Dana only while a Support Access Session (FEAT-31) is Opened for the account being viewed; once the session closes, none of this feature's screens are reachable by her | Not applicable -- outside an open session, none of FEAT-09's screens render for Dana at all |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View invoice list (FEAT-09.SPEC-001) | Nadia (Freelancer) | Always, for any project or client she owns | -- |
| View invoice list (FEAT-09.SPEC-001) | Owen (Client Primary Contact) | Own-only -- his own company's invoices across all its projects with this freelancer | -- |
| View invoice list (FEAT-09.SPEC-001) | Priya (Client Reviewer Contact) | Never | The "Invoices" entry point is not shown anywhere in Priya's portal navigation; a direct link redirects to her portal home with no error message |
| View invoice list (FEAT-09.SPEC-001) | Dana (Support Operator) | Only inside a logged, Opened Support Access Session (FEAT-31), read-only | Outside an open session, this screen is not reachable by Dana at all |
| View invoice detail (FEAT-09.SPEC-002) | Nadia (Freelancer) | Always, for any of her invoices | -- |
| View invoice detail (FEAT-09.SPEC-002) | Owen (Client Primary Contact) | Own-only -- his own company's invoices | -- |
| View invoice detail (FEAT-09.SPEC-002) | Priya (Client Reviewer Contact) | Never | A direct link to this screen redirects to her portal home with no invoice content ever rendered |
| View invoice detail (FEAT-09.SPEC-002) | Dana (Support Operator) | Only inside a logged, Opened Support Access Session, read-only | Every action control (download, pay, credit note, off-platform payment, refund) is not rendered; content is visible, actions are not |
| Issue an ad-hoc invoice or credit note (FEAT-09.SPEC-003) | Nadia (Freelancer) | Always | -- |
| Issue an ad-hoc invoice or credit note (FEAT-09.SPEC-003) | Owen, Priya | Never | This screen has no entry point in either contact's portal; a direct link redirects to portal home |
| Issue an ad-hoc invoice or credit note (FEAT-09.SPEC-003) | Dana (Support Operator) | Never | This screen is never rendered inside a support session -- issuance is excluded from read-only support access entirely (XBR-29) |
| Follow the pay link | Owen (Client Primary Contact) | Own-only, and only when FEAT-09.SPEC-009's availability rule shows the "ready to pay" state | Rendered as a disabled or absent action in the two non-ready states, per FEAT-09.SPEC-009's own wording -- never a bare "denied" |
| Follow the pay link | Nadia, Priya, Dana | Never | Not shown; the pay link is Owen's action alone |
| Download a copy of an invoice, credit note, or receipt | Nadia (Freelancer), Owen (Client Primary Contact) | Always, for any invoice each is otherwise entitled to view | -- |
| Download a copy of an invoice, credit note, or receipt | Priya, Dana | Never | Not rendered for Priya (no access at all); not rendered for Dana (read-only support access excludes downloads, per XBR-29) |

## Defaults and Derivations

Not applicable to this spec -- no field defaults or derivations are governed here; see FEAT-09.SPEC-007.

## Business Rules

- XBR-08: role entitlements follow the Access Matrix everywhere -- only Primary contacts see, pay, and download invoices; Reviewer contacts never see invoice content. This spec is the FEAT-09-specific instantiation of that rule for the Invoice entity.
- XBR-09: client isolation applies to every invoice screen -- Owen never sees another client company's invoices, and an out-of-scope link shows a plain explanation rather than another company's data.
- XBR-29: Dana's support sessions are read-only in every feature, exclude downloads and any send/pay/issue action, are always announced to Nadia by email, and are always listed in her trail -- this spec's Dana rows are the FEAT-09 instantiation of that rule.
- Every denied action across this spec's table states the exact experience (control hidden, redirect, or absent action) -- "access denied" alone is never shown to any role.

## Edge Cases

- **Priya is given a direct deep link to a specific invoice by someone who has one** -- The link resolves to a redirect to her portal home; no invoice content, not even the invoice number, is ever rendered in the process.
- **Owen's role is changed from Primary to Reviewer (a hypothetical the product does not define, since role changes are Nadia-only per FEAT-18) while he has an invoice detail screen open** -- Not applicable to this product: FEAT-18's Authorization Rules gate role changes to Nadia only, and a contact's own role change takes effect for future actions only; this spec assumes the role in effect at the moment of each access check.
- **A Support Access Session closes while Dana has an invoice detail screen open** -- The screen becomes unreachable immediately; any further interaction attempt is treated as unauthenticated for that session, consistent with FEAT-31's session-closing behavior.
- **Owen requests to download a copy of an invoice that has since been corrected (status: Corrected)** -- Allowed: the download control remains available on the original invoice's detail screen even after correction, since the original record itself is never deleted or hidden, only marked Corrected (FEAT-09.SPEC-008).
- **Dana attempts to reach FEAT-09.SPEC-003 by a direct link during an open support session** -- The screen is never rendered for her regardless of the link; issuance has no support-session-accessible form at all.

## Acceptance Criteria

**FEAT-09.SPEC-006-AC-01:** Given Nadia is signed in, when she opens any of her projects' invoice lists, then she sees every invoice regardless of status.

**FEAT-09.SPEC-006-AC-02:** Given Owen is signed into his portal, when he opens "Invoices", then he sees only his own company's invoices across all of its projects.

**FEAT-09.SPEC-006-AC-03:** Given Priya is signed into her portal, when she looks for any invoice-related entry point, then none exists anywhere in her navigation.

**FEAT-09.SPEC-006-AC-04:** Given Priya is given a direct link to a specific invoice, when the link resolves, then she is redirected to her portal home with no invoice content rendered.

**FEAT-09.SPEC-006-AC-05:** Given Dana opens a Support Access Session for a freelancer's account, when she navigates to that account's invoices, then she sees full content read-only with no action controls.

**FEAT-09.SPEC-006-AC-06:** Given Dana has no open Support Access Session, when she attempts to reach any invoice screen, then none is reachable.

**FEAT-09.SPEC-006-AC-07:** Given Nadia opens FEAT-09.SPEC-003, when the screen loads, then she can issue an ad-hoc invoice or credit note.

**FEAT-09.SPEC-006-AC-08:** Given Owen attempts to reach FEAT-09.SPEC-003 by a direct link, when the link resolves, then he is redirected to his portal home.

**FEAT-09.SPEC-006-AC-09:** Given Dana is inside an open support session, when she attempts to reach FEAT-09.SPEC-003 by any means, then it is never rendered.

**FEAT-09.SPEC-006-AC-10:** Given Owen's invoice has a ready, connected payment account, when he views its detail, then he can follow the pay link.

**FEAT-09.SPEC-006-AC-11:** Given Owen's invoice does not yet have a ready payment account, when he views its detail, then no active pay-link action is offered to him.

**FEAT-09.SPEC-006-AC-12:** Given Nadia or Owen views an invoice each is entitled to see, when they tap Download, then the copy is produced.

**FEAT-09.SPEC-006-AC-13:** Given Dana is inside an open support session, when she looks for a Download control on any invoice, then none is rendered.

**FEAT-09.SPEC-006-AC-14:** Given a Support Access Session closes while Dana has an invoice screen open, when she attempts any further interaction, then the screen is no longer reachable.

**FEAT-09.SPEC-006-AC-15:** Given an invoice has been corrected (status: Corrected), when Owen or Nadia attempts to download the original, then the download still succeeds, since the original record is never hidden by correction.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- owned by FEAT-09.SPEC-007) | 0 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 0 (N/A -- owned by FEAT-09.SPEC-007) | 0 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Invoice Content, Numbering, Amount & Due-Date Rules

## Overview

**Name:** Invoice Content, Numbering, Amount & Due-Date Rules
**ID:** FEAT-09.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs sequential numbering, required business/billing details, amount-matches-trigger validation, and default-payment-terms due-date derivation for every invoice, whether automatically generated or manually issued.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending
**Governed Entity:** Invoice (specifically the `invoice_number`, `amount`, `currency`, `tax_label`, `tax_rate`, `total`, business/billing details, `issue_date`, and `due_date` fields)

## Scope and Non-Goals

**In Scope:**
- Sequential invoice numbering, unique per freelancer, across both automatic and manual creation paths
- The required-content gate (freelancer business details, client billing details) that blocks sending
- The amount-matches-trigger rule and its application to ad-hoc invoices and credit notes
- Due-date derivation from `default_payment_terms`, and Nadia's ability to adjust it before sending

**Non-Goals:**
- Currency selection and tax-rate computation -- owned entirely by Currency & Tax Handling (FEAT-15, XBR-17); this spec applies whatever currency and tax line FEAT-15 has already configured for the project, and performs no calculation of its own
- Who may view, issue, or act on an invoice -- owned by FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules)
- Immutability once sent and the mechanics of a correction -- owned by FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules); this spec defines what content a correction carries, not whether it is permitted
- Automatic tax calculation per country or region -- excluded per scope-boundaries.md (SC-16): the tax line is a freelancer-configured label and rate, never computed by this product

## Governed Entity

**Entity:** Invoice
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| invoice_number | text (derived) | Unique and sequential per freelancer |
| amount | number | The amount matching the triggering milestone/deposit/completion price, or the entered ad-hoc/credit amount |
| currency | text | Set from the project's configured currency (FEAT-15) |
| tax_label, tax_rate | text / number | Set from the project's configured tax line (FEAT-15) |
| total | number | amount plus the tax line applied at `tax_rate` |
| freelancer business details | text (name, address, tax ID) | Read from the Freelancer Account (FEAT-21) |
| client billing details | text (billing name, billing address, optional tax ID) | Read from the Client record (FEAT-01) |
| issue_date | date | Set to the moment of generation or manual issuance |
| due_date | date | Derived from `default_payment_terms`, adjustable before sending |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-004 | Automatic Invoice Generation | Applied inline at creation, before the send hand-off |
| FEAT-09.SPEC-005 | Manual Invoice & Credit Note Recording | Applied inline at creation, for both the ad-hoc and credit-note paths |
| FEAT-09.SPEC-003 | Manual Invoice & Credit Note Issuance | Referenced for field-level validation and the due-date pre-fill/override on the issuance form |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| invoice_number | Must be the next unused sequential number for this freelancer; never reused, never skipped | Always, at creation | On create (either path) | Not user-facing -- this field is never entered by a human; a numbering conflict is a system-level retry condition, not a validation error shown to Nadia | Yes (system-level) |
| amount | Must equal the triggering event's price (automatic path) or be a positive value entered by Nadia (manual path) | Always | On create (automatic); on blur and submit (manual, via FEAT-09.SPEC-003) | "Enter an amount greater than zero." (manual path only -- the automatic path has no user-facing entry point for this field) | Yes |
| currency | Must be the project's configured currency; a project with no configured currency blocks generation entirely | Always | On create (either path) | "This project's currency isn't set yet. Set it before the first invoice." | Yes |
| tax_label, tax_rate | Must be the project's configured tax line, or explicitly none if the freelancer has configured no tax line | Always | On create (either path) | Not applicable -- absence of a tax line is a valid configuration, not an error | No |
| total | Must equal amount plus the tax line applied at tax_rate, computed once at creation and never independently entered | Always | On create (either path) | Not user-facing -- `total` is always derived, never entered | Yes (system-level) |
| freelancer business details | Must be complete (name, address; tax ID optional) before any invoice for this freelancer can be sent | Before the first invoice, and every subsequent one | On create (either path) | "Your business details aren't complete yet. Add them in Settings before this invoice can be sent." | Yes |
| client billing details | Must be complete (billing name, billing address; tax ID optional) before this client's first invoice, and every subsequent one, can be sent | Always | On create (either path) | "This client's billing details are incomplete. Add a billing name and address before sending an invoice." | Yes |
| issue_date | Set automatically to the moment of creation; never entered by a human | Always | On create (either path) | Not applicable -- no user input | No |
| due_date | Must derive from `default_payment_terms` unless Nadia overrides it before sending (manual path only) | Always | On create (automatic); on the issuance form, adjustable before submit (manual) | "Choose a due date." (only if Nadia clears the pre-filled value on the manual form) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Amount-matches-trigger | amount, total, triggering_event | On the automatic path, `amount` must exactly equal the price recorded on the triggering milestone, deposit, or completion payment at the moment of the trigger -- no rounding or adjustment | Not user-facing -- a mismatch on this path is a defect in the upstream trigger data, not a condition a user corrects; FEAT-09.SPEC-004 treats it as a generation-blocking system condition |
| Total-derivation | amount, tax_rate, total | `total` is always `amount` plus (`amount` × `tax_rate`), computed once at creation | Not applicable -- always derived |
| Credit-amount-within-original | amount (credit note), total (original invoice) | A credit note's amount can never exceed the original invoice's total | "A credit note cannot exceed the original invoice's total." |
| Billing-completeness-both-sides | freelancer business details, client billing details | Both sets must be complete; either one missing blocks sending, and each is reported by name so the freelancer knows which side to fix | See the two Field Validation Rules rows above -- the two messages are distinct so the freelancer is never told to fix the wrong side |

## Authorization Rules

Not applicable to this spec -- who may issue, view, or act on an invoice is governed entirely by FEAT-09.SPEC-006. This spec governs content correctness, not access.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| invoice_number | Next sequential integer after the highest ever assigned to this freelancer's invoices, across both automatic and manual creation | On create | No |
| issue_date | Current date and time at the moment of creation | On create | No |
| due_date | `issue_date` plus the freelancer's `default_payment_terms` (e.g., due on receipt, or within N days) | On create | Yes -- only on the manual-issuance form (FEAT-09.SPEC-003); the automatic path (FEAT-09.SPEC-004) always applies the derivation with no override step, since there is no freelancer interaction at generation |
| total | `amount` plus (`amount` × `tax_rate`), or `amount` alone if no tax line is configured | On create | No |
| tax_label, tax_rate | The project's configured tax line (FEAT-15), or none if unconfigured | On create | No -- Nadia changes this only through Currency & Tax Handling (FEAT-15), not on any FEAT-09 screen |

## Business Rules

- XBR-16: every invoice carries a due date from the freelancer's default payment terms (adjustable before sending), a unique sequential number, her business details, and the client's billing details; sending is blocked until both detail sets exist. This spec is the authoritative source of that gate for FEAT-09.
- XBR-17: currency and tax line are set once per project before the first invoice and applied exactly as configured; this spec never computes tax, only applies FEAT-15's configuration.
- Invoice numbering is a single sequence per freelancer that spans both the automatic (FEAT-09.SPEC-004) and manual (FEAT-09.SPEC-005) creation paths -- there is no separate numbering series for ad-hoc invoices or credit notes.
- A credit note receives its own new sequential number; it is never treated as a modification of the original invoice's number (this rule interacts with, and is reinforced by, FEAT-09.SPEC-008's immutability guarantee).

## Edge Cases

- **Two invoices for the same freelancer are created at effectively the same instant by different triggers (e.g., a milestone approval and an ad-hoc issuance)** -- Both draw from the same sequential numbering source; each is assigned a distinct, consecutive number with no gap and no collision, consistent with numbering being a single, freelancer-wide sequence rather than per-project.
- **A freelancer's business details are completed for the first time moments before a queued generation runs** -- The billing-completeness check reads current state at creation time, not at the moment the trigger originally fired, so a just-completed detail set is honored and generation proceeds.
- **The freelancer's default_payment_terms changes after an invoice's due_date has already been derived** -- The derivation is a one-time snapshot at creation; a later change to `default_payment_terms` never retroactively alters an already-created invoice's `due_date`.
- **Nadia enters a due date on the manual form that is earlier than the issue date** -- Accepted: the product defines no minimum-lead-time rule for a due date (a "due on receipt" term legitimately produces a due date equal to the issue date), so an earlier or same-day due date is valid.
- **A project's tax line is explicitly configured as "none"** -- `tax_label`/`tax_rate` are absent, `total` equals `amount` exactly, and the invoice's content correctly omits a tax line rather than showing a zero-rate line, which the product treats as a distinct, valid configuration.
- **The client's tax_id is left blank** -- Never blocks anything; `tax_id` is always optional on both sides of the billing-completeness gate.

## Acceptance Criteria

**FEAT-09.SPEC-007-AC-01:** Given a freelancer has issued four prior invoices numbered 1-4, when a fifth is created by either path, then it is numbered 5, with no gap and no reuse.

**FEAT-09.SPEC-007-AC-02:** Given a milestone approval triggers an invoice for a price of $500, when FEAT-09.SPEC-004 creates it, then the invoice's `amount` is exactly $500 plus the project's configured tax line.

**FEAT-09.SPEC-007-AC-03:** Given Nadia enters $0 as the amount on the manual-issuance form, when she attempts to submit, then she sees "Enter an amount greater than zero." and submission is blocked.

**FEAT-09.SPEC-007-AC-04:** Given a project has no configured currency, when a trigger attempts to generate its first invoice, then generation is blocked with "This project's currency isn't set yet. Set it before the first invoice."

**FEAT-09.SPEC-007-AC-05:** Given the Freelancer Account's business details are incomplete, when any invoice for that freelancer is created, then sending is blocked with "Your business details aren't complete yet. Add them in Settings before this invoice can be sent."

**FEAT-09.SPEC-007-AC-06:** Given the Client's billing details are incomplete, when any invoice for that client is created, then sending is blocked with "This client's billing details are incomplete. Add a billing name and address before sending an invoice."

**FEAT-09.SPEC-007-AC-07:** Given both business and billing details are complete, when an invoice is created, then it proceeds to `issue_date` and `due_date` assignment with no block.

**FEAT-09.SPEC-007-AC-08:** Given Nadia's default payment terms is "due within 14 days" and an invoice is generated automatically, when it is created, then `due_date` is set to 14 days after `issue_date` with no freelancer interaction.

**FEAT-09.SPEC-007-AC-09:** Given Nadia is issuing an ad-hoc invoice, when the form pre-fills the due date from her default terms, then she can adjust it to any other date before submitting, and the adjusted date is what gets recorded.

**FEAT-09.SPEC-007-AC-10:** Given Nadia clears the pre-filled due date on the manual form without entering another, when she attempts to submit, then she sees "Choose a due date." and submission is blocked.

**FEAT-09.SPEC-007-AC-11:** Given a credit note's entered amount exceeds the original invoice's total, when Nadia submits it, then she sees "A credit note cannot exceed the original invoice's total." and it is not recorded.

**FEAT-09.SPEC-007-AC-12:** Given a credit note's entered amount exactly equals the original invoice's total, when Nadia submits it, then it is accepted.

**FEAT-09.SPEC-007-AC-13:** Given a project's tax line is configured as none, when an invoice is created, then its `total` equals `amount` exactly, with no tax line shown.

**FEAT-09.SPEC-007-AC-14:** Given the client's tax_id is blank, when billing completeness is evaluated, then it never factors into the block.

**FEAT-09.SPEC-007-AC-15:** Given Nadia's default_payment_terms changes after an invoice was already created under the old terms, when she views that invoice, then its `due_date` remains exactly as originally derived, unaffected by the later change.

**FEAT-09.SPEC-007-AC-16:** Given Nadia enters a due date on the manual form that is the same day as the issue date, when she submits, then it is accepted as valid.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 9 | 9 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 0 (N/A -- owned by FEAT-09.SPEC-006) | 0 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Invoice Immutability & Correction Rules

## Overview

**Name:** Invoice Immutability & Correction Rules
**ID:** FEAT-09.SPEC-008
**Type:** Logic/Rule
**Purpose:** Enforces that a sent invoice is never silently edited and that a correction is always a visible credit note or a new invoice.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending
**Governed Entity:** Invoice (specifically the `status` field's immutability boundary and the correction relationship it creates)

## Scope and Non-Goals

**In Scope:**
- The boundary at which an invoice becomes immutable (the moment `status` moves to `Sent` or later)
- The rule that a correction is always a new, linked Invoice record (a credit note), never an in-place edit
- Marking the original invoice `Corrected` at the exact moment its credit note is recorded
- What every screen that would otherwise offer an edit control must show instead once an invoice is Sent or later

**Non-Goals:**
- Deciding whether Nadia is authorized to issue a correction at all -- owned by FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules)
- The content and numbering a credit note carries -- owned by FEAT-09.SPEC-007
- Payment-status transitions (Paid, Overdue, Refunded, Disputed) -- owned by FEAT-10, FEAT-11, and FEAT-25 respectively; this spec governs only the Generated/Sent/Corrected transitions this feature itself owns
- Deleting or archiving an invoice -- excluded per the Brief's Entity-Lifecycle Coverage Matrix: this feature never deletes or archives an Invoice; deletion is owned exclusively by account deletion (FEAT-24), subject to legal financial-record retention (scope-boundaries.md, SC-24)

## Governed Entity

**Entity:** Invoice
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum | This spec governs the Generated -> Sent -> Corrected transitions; Payment pending, Paid, Overdue, Refunded, Partially refunded, and Disputed are owned by FEAT-10, FEAT-11, and FEAT-25 |
| Any content field once Sent+ (amount, currency, tax line, business/billing details, issue_date, due_date, invoice_number) | -- | Frozen the moment `status` reaches `Sent` -- no further write path exists on any field once sent, regardless of role |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-002 | Invoice Detail | No edit control is ever rendered once `status` is Sent or later; only "Correct with a credit note" is shown |
| FEAT-09.SPEC-003 | Manual Invoice & Credit Note Issuance | The correction entry point -- always produces a new record, never an edit form against the original |
| FEAT-09.SPEC-005 | Manual Invoice & Credit Note Recording | Writes the credit note as a new record and sets the original's `status` to `Corrected` atomically |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status (and every content field once Sent+) | No content field may be written once `status` is `Sent` or later, by any role, through any spec in this feature | Always, once `status` reaches Sent | On every attempted write | Not user-facing as a form error -- enforced structurally by the absence of any edit control (FEAT-09.SPEC-002) rather than a rejected save | Yes (structural) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Correction-is-a-new-record | status (original), the credit note's own fields | A correction is only ever expressed as a new Invoice record (a credit note) linked back to the original; the original's own content fields are never touched | Not applicable -- the product offers no path that would produce this error, since no edit control exists once Sent |
| Original-marked-Corrected-atomically | status (original), the credit note's creation | The original's `status` transitions to `Corrected` in the same atomic step that creates its credit note -- there is no window in which a credit note exists but the original still reads `Sent` | "This invoice was already corrected. View the existing credit note." (shown only on a second concurrent attempt, per FEAT-09.SPEC-005's re-check) |

## Authorization Rules

Not applicable to this spec -- who may issue a correction is governed by FEAT-09.SPEC-006. This spec governs the mechanics and boundary of immutability, not who may act on it.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| status | Set to `Generated` at creation, then `Sent` immediately once the send hand-off begins (FEAT-09.SPEC-004/FEAT-09.SPEC-005); set to `Corrected` only when a credit note is recorded against it | On create, then on the corresponding transition | No -- these three transitions are entirely system-driven; no screen offers a manual status-setting control for them |

## Business Rules

- XBR-04: evidence records -- including a sent invoice -- are never silently altered; changes happen only as new, logged events. This spec is the FEAT-09-specific instantiation of that rule for the Invoice entity.
- BRIEF.md's Constraints (record immutability): "payments and records must be correct," and a correction must be visible rather than a silent rewrite -- this spec's credit-note-as-new-record rule is the direct mechanism delivering that guarantee.
- A `Corrected` original and its credit note are permanently linked; the link is never removed, even if the credit note is itself later referenced by a refund (FEAT-25) recorded against the paid history of either record.
- Immutability applies uniformly regardless of how the invoice was created -- an automatically generated invoice (FEAT-09.SPEC-004) and a manually issued one (FEAT-09.SPEC-005) become immutable at exactly the same point: the moment `status` reaches `Sent`.

## Edge Cases

- **Nadia attempts to correct an invoice that is still in `Generated` status (has not yet actually sent, e.g., during a retried send hand-off)** -- Not applicable in practice: FEAT-09.SPEC-004 and FEAT-09.SPEC-005 both set `status: Sent` in the same automation run that creates the record, before any user could view or act on it in a `Generated`-only state; there is no user-reachable window in which an invoice is both created and still correctable as if unsent.
- **A credit note is itself later corrected** -- Permitted: a credit note is an Invoice record like any other once `status` reaches `Sent`, so it becomes immutable and correctable by the same rule -- a correction of a correction is simply another new, linked record, with the chain fully traceable through each record's link back to its predecessor.
- **Two sessions attempt to correct the same original invoice concurrently** -- Resolved by FEAT-09.SPEC-005's atomic write-time re-check (Cross-Field Rules, Original-marked-Corrected-atomically): the first to complete succeeds, and the second is refused with "This invoice was already corrected. View the existing credit note."
- **An invoice is Sent and later Paid, then Nadia issues a credit note against it** -- Allowed: the payment history (owned by FEAT-10) is untouched by the correction; the original shows both its Paid history and its Corrected status side by side, and any actual refund is a separate action Nadia takes through FEAT-25.
- **A downstream failure occurs after the original is marked Corrected but before the credit note's own send completes** -- The `Corrected` status on the original and the credit note's existence are not rolled back, since both are already evidentiary per XBR-04; only the credit note's own send is retried (FEAT-09.SPEC-010), consistent with FEAT-09.SPEC-005's failure handling.

## Acceptance Criteria

**FEAT-09.SPEC-008-AC-01:** Given an invoice's status is Sent, when Nadia views its detail, then no edit control is shown for any field.

**FEAT-09.SPEC-008-AC-02:** Given an invoice's status is Sent, when Nadia wants to correct it, then the only available action is "Correct with a credit note."

**FEAT-09.SPEC-008-AC-03:** Given Nadia issues a credit note against a Sent invoice, when it is recorded, then a new, separate Invoice record is created and the original's status becomes Corrected in the same step.

**FEAT-09.SPEC-008-AC-04:** Given an invoice has been marked Corrected, when Nadia or Owen views its original content, then every original field (amount, dates, business/billing details, invoice number) remains exactly as first sent.

**FEAT-09.SPEC-008-AC-05:** Given two sessions attempt to correct the same invoice at effectively the same time, when both reach the write, then exactly one succeeds and the other is told "This invoice was already corrected. View the existing credit note."

**FEAT-09.SPEC-008-AC-06:** Given a credit note itself reaches Sent status, when Nadia later wants to correct it, then the same immutability and correction rule applies -- a new record links back to it.

**FEAT-09.SPEC-008-AC-07:** Given an invoice was created automatically (FEAT-09.SPEC-004), when it reaches Sent status, then it becomes immutable on exactly the same terms as a manually issued invoice.

**FEAT-09.SPEC-008-AC-08:** Given a Paid invoice is later corrected by a credit note, when Nadia views its detail, then both its Paid payment history and its Corrected status are shown together, unaltered by each other.

**FEAT-09.SPEC-008-AC-09:** Given the original invoice is marked Corrected, when Nadia or Owen looks for its credit note, then a link to the linked credit note is present on the original's detail view.

**FEAT-09.SPEC-008-AC-10:** Given a correction chain exists (an invoice corrected by a credit note, which is itself later corrected), when any record in the chain is viewed, then its link back to its immediate predecessor is traceable.

**FEAT-09.SPEC-008-AC-11:** Given the credit note's own send fails after the original was already marked Corrected, when the failure occurs, then the Corrected status and the credit note record are unaffected, and only the send is retried.

**FEAT-09.SPEC-008-AC-12:** Given no invoice in this product is ever created without immediately reaching Sent status within the same automation run, when any invoice is inspected, then it is never observed sitting in a user-reachable Generated-only, still-editable state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 0 (N/A -- owned by FEAT-09.SPEC-006) | 0 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Pay-Link Availability & No-Account Fallback Rule

## Overview

**Name:** Pay-Link Availability & No-Account Fallback Rule
**ID:** FEAT-09.SPEC-009
**Type:** Logic/Rule
**Purpose:** Governs what an invoice's pay link shows and how it is worded when Nadia has no connected, ready payment account, or when a previously working connection later needs attention or disconnects.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending
**Governed Entity:** Invoice (specifically the `pay_link availability` field, derived from Payment Account Connection)

## Scope and Non-Goals

**In Scope:**
- The three pay-link states an invoice can show (ready to pay, online payment not yet available, temporarily unavailable) and the exact wording for each
- Deriving the current state from the Payment Account Connection's status at the moment each invoice screen or email renders it
- The behavior when a connection that was working later needs attention or disconnects, on invoices already open

**Non-Goals:**
- Connecting, reconnecting, or disconnecting the payment account itself -- owned entirely by Payment Account Connection (FEAT-32); this spec only reads its status
- The actual payment flow once the pay link is followed -- owned by Invoice Payment Processing (FEAT-10)
- Recording a payment received outside the pay link -- owned by FEAT-10's off-platform recording capability; this spec only governs what the link itself shows before that happens

## Governed Entity

**Entity:** Invoice
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| pay_link availability | derived | The wording state shown on the invoice's pay link, derived from Payment Account Connection at render time |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-002 | Invoice Detail | Renders the pay-link status banner on every view |
| FEAT-09.SPEC-004 | Automatic Invoice Generation | Checks availability at generation time and carries the resulting wording state onto the new invoice |
| FEAT-09.SPEC-010 | Invoice Issued & Copy Confirmation Notification | Renders the same three states, worded identically, in the invoice email to Owen |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| pay_link availability | Must be one of exactly three states: Ready, Not yet available, Temporarily unavailable -- derived, never entered by any user | Always | Evaluated fresh at every render (screen view or email composition), never cached from generation time | Not applicable -- this is a derived display state, not a user-facing validation error | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Connection-status-drives-wording | pay_link availability, Payment Account Connection status | `Ready` when the connection's status is Connected; `Not yet available` when the status is Not connected; `Temporarily unavailable` when the status is Needs attention or Disconnected after having been previously connected | Not applicable -- these are display states, not blocking errors; each has its own exact wording, defined below |

## Authorization Rules

Not applicable to this spec -- who may follow the pay link at all is governed by FEAT-09.SPEC-006. This spec governs only what the link says once a role that may follow it (Owen) is looking at it.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| pay_link availability | Ready (Connection: Connected) -> exact wording "Ready to pay"; Not connected -> "Online payment isn't set up yet. [Freelancer name]'s direct payment instructions are below." with Nadia's stated alternative instructions; Needs attention or Disconnected (previously connected) -> "Online payment is temporarily unavailable. Please try again shortly, or contact [Freelancer name] directly." | Recalculated at every render, never stored as a frozen value on the invoice | No -- entirely derived from the live Payment Account Connection status; neither Nadia nor Owen can set it directly |

## Business Rules

- XBR-19: with no connected payment account, invoices still issue and send with instructions for paying the freelancer directly; when the connection needs attention, pay links show "online payment is temporarily unavailable"; disconnecting warns that open invoices lose their pay links. This spec is the FEAT-09-side instantiation of that rule.
- FEAT-32 (Payment Account Connection) is the sole owner and authority for the underlying connection status; this spec never writes to Payment Account Connection, only reads it (per the dependency map's External Touchpoints table: "FEAT-09 validated with no Integration spec for this capability: it derives pay-link availability from the connection status").
- The wording is identical wherever it appears -- FEAT-09.SPEC-002's banner and FEAT-09.SPEC-010's email render the same three states with the same text, per the Brief's Shared UI Patterns ("Pay-link status banner").
- An invoice generated while the connection is Not connected still issues and sends per FEAT-09.SPEC-004 -- this spec never blocks generation or sending; it only changes what the pay link says.

## Edge Cases

- **Nadia connects her payment account after several invoices were already sent with "not yet available" wording** -- Every open invoice's pay-link banner re-derives to Ready the next time it renders (screen view or a re-sent email is not triggered automatically); Owen sees "Ready to pay" the next time he opens any of those invoices, with no action required from Nadia beyond connecting.
- **The connection moves from Connected to Needs attention while Owen has an invoice detail screen open with the pay link visible** -- The banner updates in place to "Online payment is temporarily unavailable..." consistent with FEAT-09.SPEC-002's live-status-update behavior; Owen never completes a payment against a connection that has just become unready, since FEAT-10's own pay flow re-checks status at the moment of payment.
- **Nadia disconnects her account entirely while several invoices are open and unpaid** -- Each open invoice's pay link switches to the "temporarily unavailable" wording (XBR-19), and Nadia sees FEAT-32's own disconnect warning that open invoices will lose their pay links until she reconnects.
- **An invoice is Paid before the connection ever needed attention** -- The pay-link banner becomes moot once paid (FEAT-10 owns the Paid-state display); this spec's wording is never shown on an invoice already marked Paid.
- **Owen opens an invoice email sent while the connection was Ready, but the connection has since moved to Needs attention** -- The email's static content is not re-rendered after sending, but the live invoice detail page it links to (FEAT-09.SPEC-002) always reflects the current status; Owen sees the up-to-date "temporarily unavailable" wording once he opens the linked page, even though the email text itself is fixed at send time.
- **The connection has never existed at all (freelancer has never started the connect flow)** -- Treated identically to "Not connected" -- the "not yet available" wording and Nadia's direct-payment instructions apply from the very first invoice.

## Acceptance Criteria

**FEAT-09.SPEC-009-AC-01:** Given Nadia has a Connected, ready payment account, when an invoice is generated, then its pay link shows "Ready to pay."

**FEAT-09.SPEC-009-AC-02:** Given Nadia has never connected a payment account, when an invoice is generated and sent, then it shows "Online payment isn't set up yet." with her direct-payment instructions, and it still sends.

**FEAT-09.SPEC-009-AC-03:** Given Nadia's connection status is Needs attention, when Owen opens an open invoice, then he sees "Online payment is temporarily unavailable. Please try again shortly, or contact [Freelancer name] directly."

**FEAT-09.SPEC-009-AC-04:** Given Nadia's connection status is Disconnected after having previously been connected, when Owen opens an open invoice, then he sees the same "temporarily unavailable" wording as the Needs-attention state.

**FEAT-09.SPEC-009-AC-05:** Given Nadia connects her account after invoices were sent with "not yet available" wording, when Owen next opens any of those invoices, then the banner shows "Ready to pay" with no further action from Nadia.

**FEAT-09.SPEC-009-AC-06:** Given the connection changes from Connected to Needs attention while Owen has an invoice detail screen open, when the change lands, then the banner updates in place to the temporarily-unavailable wording.

**FEAT-09.SPEC-009-AC-07:** Given Nadia disconnects her account while invoices remain open and unpaid, when the disconnect completes, then every open invoice's pay link switches to "temporarily unavailable" wording.

**FEAT-09.SPEC-009-AC-08:** Given an invoice has already been marked Paid, when its detail is viewed, then the pay-link availability wording from this spec is not shown.

**FEAT-09.SPEC-009-AC-09:** Given Owen opens an invoice email sent while the connection was Ready but the connection has since changed, when he follows the link to the live invoice detail page, then he sees the current, up-to-date wording rather than the state at send time.

**FEAT-09.SPEC-009-AC-10:** Given Nadia has never started the connect flow at all, when her first invoice is generated, then it is treated identically to "Not connected" with the same wording and instructions.

**FEAT-09.SPEC-009-AC-11:** Given FEAT-09.SPEC-010 composes the invoice email at the same moment FEAT-09.SPEC-002 would render the detail banner, when both render, then they show the exact same wording for the same connection state.

**FEAT-09.SPEC-009-AC-12:** Given this spec is asked for a pay-link state and the connection status is any value other than Connected, Not connected, Needs attention, or Disconnected, then no such value exists in the product's definition of Payment Account Connection, so this condition never arises.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 0 (N/A -- owned by FEAT-09.SPEC-006) | 0 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Notification Spec: Invoice Issued & Copy Confirmation Notification

## Overview

**Name:** Invoice Issued & Copy Confirmation Notification
**ID:** FEAT-09.SPEC-010
**Type:** Notification
**Purpose:** Emails Owen the invoice with its pay link and emails Nadia a confirmation copy, whenever any invoice -- automatic, ad hoc, or credit note -- is sent.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- The invoice email to Owen, carrying the pay link in whichever of the three availability states applies
- The confirmation copy to Nadia, sent alongside every invoice send
- Retry and delivery-failure behavior for both recipients
- The identical treatment of automatic invoices, ad-hoc invoices, and credit notes -- all three trigger this same notification

**Non-Goals:**
- The pay-link wording itself -- owned by FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule); this spec only renders that spec's output into the email
- Overdue payment reminders after this initial send -- owned entirely by Automated Payment Reminders (FEAT-11), a distinct notification with its own schedule
- The underlying email send and delivery/bounce status mechanism -- owned by the Transactional Email Delivery capability (FEAT-14.SPEC-001); this spec composes the message and consumes that capability's delivery status, but does not implement the sending itself
- Notifying Priya -- excluded per the Access Matrix: invoice content, including its existence, is never communicated to Reviewer contacts

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, for both Owen and Nadia, on every invoice send | Email is the only channel that reaches client contacts (BRIEF.md, Ecosystem & Integrations: "Clients will not install an app"), and this notification's whole purpose is to hand Owen a way to pay and Nadia a record that it went out |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An invoice reaches Sent status | FEAT-09.SPEC-004 (Automatic Invoice Generation) | Fires immediately once an automatically generated invoice's send hand-off begins | Invoice reference, all its content fields, pay-link availability state |
| An invoice or credit note reaches Sent status | FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording) | Fires immediately once a manually issued invoice or a credit note's send hand-off begins | Invoice or credit-note reference, all its content fields, pay-link availability state |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) receives the invoice email for his own company's invoices, own-only, per the Access Matrix. Nadia (Freelancer) receives the confirmation copy for every invoice she issues, automatically or manually. Priya (Client Reviewer Contact) never receives this notification, per the Access Matrix's total exclusion of Reviewer contacts from invoice content.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A | -- | Always on | -- |

This is a transactional record email core to the financial relationship (an invoice and its own confirmation copy); per XBR-30, transactional emails central to the record always send and cannot be switched off by either recipient.

**Quiet Hours:** N/A -- the product defines no quiet-hours window for transactional invoice delivery (XBR-30 distinguishes only optional notifications as quiet-hours-eligible); an invoice is time-sensitive to both the freelancer's cash flow and the due-date clock it starts, so it is sent immediately regardless of the hour.

## Content Definition

**Email (to Owen):**
- **Subject:** New invoice from {freelancer_business_name}: {invoice_total} due {due_date}
- **Body:**
  Hi {owen_first_name},

  {freelancer_business_name} has sent you an invoice for {project_name}.

  Invoice {invoice_number} -- {invoice_total} -- due {due_date}

  {pay_link_status_block}

  You can view the full invoice, its status, and a downloadable copy any time from your portal.
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for this invoice
- **Secondary link:** View all invoices -- deep-links to FEAT-09.SPEC-001 (Invoice List), scoped to Owen's own company

**Email (to Nadia -- confirmation copy):**
- **Subject:** Invoice {invoice_number} sent to {client_billing_name}
- **Body:**
  Hi {nadia_first_name},

  Invoice {invoice_number} for {project_name} was sent to {client_billing_name} -- {invoice_total}, due {due_date}.

  {trigger_source_line}
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for this invoice

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_business_name} | Freelancer Account -- business_name | Nadia Reyes Design | Never empty -- FEAT-09.SPEC-007 blocks sending entirely until this field is complete |
| {owen_first_name} | Client Contact -- name (first token) | Owen | "there" (greeting renders "Hi there,") |
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | "there" |
| {project_name} | Project -- project_name | Brand Refresh -- Phase 2 | Never empty -- project_name is required at project creation (FEAT-01) |
| {invoice_number} | Invoice -- invoice_number | INV-0047 | Never empty -- always assigned before this notification fires (FEAT-09.SPEC-007) |
| {invoice_total} | Invoice -- total, formatted with currency | $2,400.00 | Never empty -- total is always derived before sending |
| {due_date} | Invoice -- due_date, in the recipient's own time zone | May 14, 2026 | Never empty -- due_date is always set before sending |
| {client_billing_name} | Client -- billing_name | Meridian Studios LLC | Never empty -- billing completeness blocks sending until this exists |
| {pay_link_status_block} | Derived -- FEAT-09.SPEC-009's current availability state, rendered as its own short paragraph plus the pay-now button when Ready | "Pay this invoice online: [Pay now]" | Renders the "not yet available" or "temporarily unavailable" paragraph instead of a button, per FEAT-09.SPEC-009; never blank |
| {trigger_source_line} | Derived -- the triggering event, phrased for Nadia's context | "Triggered by: milestone approval" / "Issued manually" / "Credit note against INV-0041" | Never empty -- every invoice has exactly one recorded triggering_event |

## Delivery Rules

**Batching:** None -- each invoice send produces exactly one email to Owen and one to Nadia; invoices are never batched into a single combined notification, since each carries its own pay link and due date that the recipient must act on individually.
**Deduplication:** At most one invoice email and one confirmation copy per invoice send. A retried send hand-off (per FEAT-09.SPEC-004/FEAT-09.SPEC-005's failure handling) never produces a second pair of emails once the first pair is confirmed delivered; a retry only re-attempts a send that has not yet succeeded.
**Retry on failure:** Delivery failure on either email is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure to Owen, the failure is surfaced to Nadia as a delivery warning on the project (XBR-30); after the final failure to Nadia's own copy, no separate warning is generated, since the invoice itself remains fully visible to her in FEAT-09.SPEC-001 and FEAT-09.SPEC-002 regardless of this email's outcome.
**Expiry:** Not applicable -- an invoice notification does not expire or become stale to withhold; if delivery is still retrying when the retry window ends, the final failure is surfaced per the Retry rule above rather than the notification being silently dropped.

## Edge Cases

- **The invoice is corrected by a credit note before Owen ever opens the original email** -- The original email's content is not retracted or altered (emails are immutable once sent); Owen's link to FEAT-09.SPEC-002 always shows the invoice's current, live status (now Corrected, with a link to the credit note), even though the email text itself still reads as it did at send time.
- **Owen's email address bounces on the first delivery attempt** -- Retried per the Retry rule; after the final failure, Nadia sees a delivery warning on the project naming Owen's contact as undeliverable, consistent with the pattern used across this pipeline's other transactional notifications.
- **The Payment Account Connection changes state between this email being composed and Owen opening it** -- The email's {pay_link_status_block} reflects the state at send time and is not retroactively updated; Owen sees the current state instead once he follows the CTA into FEAT-09.SPEC-002, per FEAT-09.SPEC-009's live-rendering behavior on that screen.
- **A credit note is sent against an invoice that was itself never successfully delivered to Owen** -- The credit note produces its own independent send attempt to Owen and its own confirmation copy to Nadia; a prior delivery failure on the original invoice does not suppress or alter the credit note's own delivery.
- **Nadia issues an ad-hoc invoice while offline, and connectivity returns moments later** -- Per FEAT-09.SPEC-003/FEAT-09.SPEC-005's offline handling, the submission itself only succeeds once connectivity returns; this notification fires only after a successful recording, so no notification is ever queued against a submission that never actually completed.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-004 (Automatic Invoice Generation) | Triggered by (inbound) | Fires this notification once an automatic invoice reaches Sent |
| FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording) | Triggered by (inbound) | Fires this notification once a manual invoice or credit note reaches Sent |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (inbound) | Supplies the exact wording for {pay_link_status_block} |
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (outbound) | Both CTAs deep-link here |
| FEAT-09.SPEC-001 (Invoice List) | Navigation (outbound) | Owen's secondary "View all invoices" link |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (inbound) | Provides the send mechanism and delivery/bounce status this spec consumes |
| FEAT-01.SPEC-005 (Project Detail) | Affects (outbound) | Shows the delivery-failure warning when Owen's copy exhausts retries |

## Analytics and Success Signals

- **invoice_email_delivered** (recipient: owen / nadia; triggering event type) -- supports success-metrics.md: "Notification Delivery Reliability"
- **invoice_email_opened** (recipient) -- supports success-metrics.md: "Notification Delivery Reliability"
- **invoice_email_cta_tapped** (recipient; destination: invoice_detail / invoice_list) -- supports success-metrics.md: "Time to Payment" (Owen's CTA tap is the first observable step toward the pay flow this metric measures)
- **invoice_email_delivery_failed** (recipient; retry_count) -- N/A -- no Stage 2 metric measures failed invoice-email delivery directly, though "Notification Delivery Reliability" measures the aggregate; retained here as this notification's own diagnostic signal, consistent with FEAT-05.SPEC-008's pattern for the same event shape

## Acceptance Criteria

**FEAT-09.SPEC-010-AC-01:** Given an invoice is generated automatically and reaches Sent status, when this notification fires, then Owen receives the invoice email and Nadia receives the confirmation copy.

**FEAT-09.SPEC-010-AC-02:** Given Nadia issues an ad-hoc invoice successfully, when it reaches Sent status, then the same two emails are sent, worded identically to the automatic path.

**FEAT-09.SPEC-010-AC-03:** Given a credit note is recorded, when it reaches Sent status, then Owen and Nadia each receive their respective emails for the credit note, distinct from the original invoice's emails.

**FEAT-09.SPEC-010-AC-04:** Given Nadia has a Connected, ready payment account, when Owen's email is composed, then {pay_link_status_block} renders the "Pay now" button.

**FEAT-09.SPEC-010-AC-05:** Given Nadia has no connected payment account, when Owen's email is composed, then {pay_link_status_block} renders the "not yet available" wording with no pay button.

**FEAT-09.SPEC-010-AC-06:** Given Priya is a Reviewer contact at the same client company as Owen, when any invoice is sent, then Priya never receives this notification.

**FEAT-09.SPEC-010-AC-07:** Given Owen's email bounces on the first delivery attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the project for Nadia.

**FEAT-09.SPEC-010-AC-08:** Given Nadia's own confirmation copy fails delivery after all retries, when the final failure occurs, then no separate project-level warning is generated, since the invoice remains fully visible to her in-product.

**FEAT-09.SPEC-010-AC-09:** Given Owen taps "View invoice" from the email, when the tap registers, then he lands on FEAT-09.SPEC-002 for that exact invoice.

**FEAT-09.SPEC-010-AC-10:** Given an invoice is corrected after its original email was sent, when Owen later opens that original email's link, then he sees the invoice's current, live Corrected status rather than the email's original static text.

**FEAT-09.SPEC-010-AC-11:** Given a send hand-off is retried after a transient failure, when the retry succeeds, then exactly one pair of emails is ultimately delivered -- never two.

**FEAT-09.SPEC-010-AC-12:** Given the invoice_email_delivered event is emitted for Owen, when it fires, then it carries the recipient and the triggering event type.

**FEAT-09.SPEC-010-AC-13:** Given no quiet-hours window applies to this notification, when an invoice is generated at any hour, then both emails send immediately with no hold.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 | 2 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
