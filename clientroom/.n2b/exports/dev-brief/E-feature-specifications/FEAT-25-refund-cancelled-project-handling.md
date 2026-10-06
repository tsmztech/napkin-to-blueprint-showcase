# FEAT-25 — Refund & Cancelled Project Handling

This chapter covers Refund & Cancelled Project Handling, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 8 specifications carrying 121 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-25.SPEC-001 | Mark Invoice Refunded Screen | screen | 18 |
| FEAT-25.SPEC-002 | Mark Project Cancelled Screen | screen | 17 |
| FEAT-25.SPEC-003 | Refund & Partial Refund Recording | automation | 14 |
| FEAT-25.SPEC-004 | Project Cancellation Recording | automation | 11 |
| FEAT-25.SPEC-005 | Payment Reversal (Chargeback) Recording | automation | 12 |
| FEAT-25.SPEC-006 | Refund, Cancellation & Reversal Authorization and Validation Rules | logic-rule | 23 |
| FEAT-25.SPEC-007 | Refund & Cancellation Notification | notification | 14 |
| FEAT-25.SPEC-008 | Payment Reversal Notification | notification | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Refund & Cancelled Project Handling

## Summary

**Feature:** Refund & Cancelled Project Handling
**ID:** FEAT-25
**Description:** The freelancer marks an invoice as refunded (the refund itself is issued through her own processor account) and can mark a project cancelled, preserving the record rather than deleting it.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Open Questions notes that "the refund, chargeback and cancelled-project flow was not defined," while Constraints requires "payments and records must be correct" and that records are never silently altered. Ranked Important rather than Core because most projects never need it; phased MVP because payments begin at MVP (FEAT-10), so a way to record a refund or cancellation correctly must exist from the same point, even in minimal form, to keep the record trustworthy. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Mark an invoice refunded — records that a refund was issued outside the platform, through the freelancer's own processor
- Mark a project cancelled — records that work has stopped, without deleting history
- Record a partial refund — mark the amount refunded when only part of an invoice is returned
- Payment reversal notice — when the payment processor reports a chargeback or reversal on a paid invoice, the invoice shows Disputed and Nadia is notified

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-25.SPEC-001 | Mark Invoice Refunded Screen | Screen | Nadia (Freelancer) | Nadia marks a paid invoice Refunded or Partially refunded, entering the refunded amount and an optional reason, from the invoice detail view |
| FEAT-25.SPEC-002 | Mark Project Cancelled Screen | Screen | Nadia (Freelancer) | Nadia marks a project Cancelled, entering an optional reason, from the project detail view, without deleting any project history |
| FEAT-25.SPEC-003 | Refund & Partial Refund Recording | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | Validates and persists Nadia's refund entry — full or partial, never exceeding the amount paid — sets the invoice to Refunded or Partially refunded, and preserves the original Paid record rather than overwriting it |
| FEAT-25.SPEC-004 | Project Cancellation Recording | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | Persists Nadia's cancellation of a project, sets `cancelled_at`, derives the Cancelled stage, and preserves every existing proposal, milestone, deliverable, and invoice record unchanged |
| FEAT-25.SPEC-005 | Payment Reversal (Chargeback) Recording | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | Applies an inbound reversal or chargeback notice relayed from the payment-processing capability to a Paid invoice, setting it Disputed alongside its preserved Paid record and marking the underlying Payment Reversed |
| FEAT-25.SPEC-006 | Refund, Cancellation & Reversal Authorization and Validation Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Governs who may mark a refund or cancellation, the refund-amount and no-partial-payment limits, the refunded-cannot-be-repaid-without-correction rule, reject-with-refresh concurrency, and Dana's view-only support access |
| FEAT-25.SPEC-007 | Refund & Cancellation Notification | Notification | Owen (Client Primary Contact), Nadia (Freelancer) | Emails Owen when an invoice he was billed is marked Refunded/Partially refunded or when his project is marked Cancelled |
| FEAT-25.SPEC-008 | Payment Reversal Notification | Notification | Nadia (Freelancer) | Emails Nadia the moment a payment reversal or chargeback is recorded, so she knows to respond in her own processor account |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Mark an invoice refunded — records that a refund was issued outside the platform, through the freelancer's own processor | FEAT-25.SPEC-001, FEAT-25.SPEC-003 | The screen collects the refund amount and optional reason; the automation validates and applies the Refunded status, preserving the original Paid record | Phase 2 (Explicit) |
| Mark a project cancelled — records that work has stopped, without deleting history | FEAT-25.SPEC-002, FEAT-25.SPEC-004 | The screen collects an optional reason; the automation sets the project Cancelled and `cancelled_at` while leaving every other project record untouched | Phase 2 (Explicit) |
| Record a partial refund — mark the amount refunded when only part of an invoice is returned | FEAT-25.SPEC-001, FEAT-25.SPEC-003 | The screen's amount field accepts less than the full paid total; the automation sets Partially refunded instead of Refunded and enforces the amount-cannot-exceed-paid limit | Phase 2 (Explicit) |
| Payment reversal notice — when the payment processor reports a chargeback or reversal on a paid invoice, the invoice shows Disputed and Nadia is notified | FEAT-25.SPEC-005, FEAT-25.SPEC-008 | The automation applies the inbound reversal event (relayed by FEAT-32's integration) to the invoice and Payment; the notification alerts Nadia immediately | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 4-5:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-25.SPEC-006 | Refund, Cancellation & Reversal Authorization and Validation Rules | Phase 5 (Rule-Constraint Discovery) | The Access, Validation & Limits, and Contention fields together produce 5+ interacting rules (Nadia-only write gate, Dana's view-only support constraint, refund-cannot-exceed-paid limit, no-partial-payment corollary from SC-17, refunded-cannot-be-repaid-without-correction, reject-with-refresh on Project and Invoice state races, processor-authoritative-over-manual on reversals) shared across SPEC-001 through SPEC-005 — past the inline-validation threshold |
| FEAT-25.SPEC-007 | Refund & Cancellation Notification | Phase 4 (Notification surfacing) | The Communications field names an email to Owen with a defined audience and two distinct triggers (refund, cancellation) — this carries delivery rules and cannot stay an inline toast |
| FEAT-25.SPEC-008 | Payment Reversal Notification | Phase 4 (Notification surfacing) | The Communications field separately names an email to Nadia on a payment reversal, with a different audience and trigger than SPEC-007, so it is discovered and specified independently rather than folded into the same notification |

## Entity-Lifecycle Coverage Matrix

**Entity: Invoice**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Invoices are created by Invoice Generation & Sending (FEAT-09); this feature never creates one | -- |
| Read (single) | FEAT-25.SPEC-001 | Loads the invoice Nadia is marking refunded, including its current status and the amount already paid | -- |
| Read (list) | N/A | Invoice list browsing belongs to FEAT-09 and the dashboard (FEAT-12), not to this feature | -- |
| Update | FEAT-25.SPEC-003, FEAT-25.SPEC-005 | SPEC-003 writes Refunded or Partially refunded; SPEC-005 writes Disputed alongside the preserved Paid record | Invoice `status` field (feature-dependency-map.md, Entity: Invoice) |
| Delete/Archive | N/A | This feature never deletes or archives an invoice — soft or hard. Deletion is owned entirely by account deletion (FEAT-24), subject to legal financial-record retention (feature-dependency-map.md, Entity: Invoice); recorded as an explicit non-goal below | -- |
| State Transition | FEAT-25.SPEC-003 (Paid → Refunded, Paid → Partially refunded), FEAT-25.SPEC-005 (Paid → Disputed, preserving Paid) | Sent → Payment pending → Paid transitions belong to FEAT-10; Overdue to FEAT-11; Generated/Sent to FEAT-09; a return from Refunded to Paid is never automatic and requires a logged correction (XBR-20) | -- |

**Entity: Project**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Projects are created by Client & Project Management (FEAT-01); this feature never creates one | -- |
| Read (single) | FEAT-25.SPEC-002 | Loads the project Nadia is marking cancelled | -- |
| Read (list) | N/A | Project list browsing belongs to FEAT-01; not owned by this feature | -- |
| Update | FEAT-25.SPEC-004 | Sets `stage` to Cancelled and stamps `cancelled_at` | Project `stage`/`cancelled_at` fields (feature-dependency-map.md, Entity: Project) |
| Delete/Archive | N/A | Cancellation is explicitly not deletion or archival: the project's full history (Proposal, Payment Schedule, Milestones, Deliverables, Invoices, Activity Log Entries) is preserved unchanged and remains readable (XBR-25). No in-product project delete exists at all — deletion happens only through account deletion (FEAT-24). There is no restore path because nothing is removed, no cascade because nothing downstream is touched, and no separate retention/purge policy because Cancelled projects are retained exactly like any other project for the life of the account; recorded as an explicit non-goal below | -- |
| State Transition | FEAT-25.SPEC-004 (→ Cancelled) | System-driven stage changes (acceptance, approval) never overwrite an explicit Cancelled (feature-dependency-map.md, Entity: Project, Contention) | -- |

**Entity: Payment**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Payments are created by Invoice Payment Processing (FEAT-10); this feature never creates one | -- |
| Read (single) | FEAT-25.SPEC-003, FEAT-25.SPEC-005 | SPEC-003 reads the paid amount to enforce the refund-cannot-exceed-paid limit; SPEC-005 reads the Payment to confirm a reversal applies to a genuinely paid invoice | -- |
| Read (list) | N/A | An invoice has at most one successful Payment (SC-17); no payment history list is owned by this feature | -- |
| Update | FEAT-25.SPEC-003, FEAT-25.SPEC-005 | SPEC-003 records the refunded amount against the Payment; SPEC-005 sets Payment status Reversed | Payment `status` field (feature-dependency-map.md, Entity: Payment) |
| Delete/Archive | N/A | Payment is never deleted or archived in-product; removed only on account deletion (FEAT-24) subject to legal retention (feature-dependency-map.md, Entity: Payment). This feature defines no delete/archive operation — recorded as an explicit non-goal below | -- |
| State Transition | FEAT-25.SPEC-005 (Succeeded → Reversed) | The refunded amount recorded by SPEC-003 does not itself change Payment status (a refund is issued outside the platform; only a processor-reported reversal changes status) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Payment Account Connection | FEAT-25.SPEC-005 | Supplies the inbound reversal/chargeback notice relayed from the payment-processing capability (FEAT-32.SPEC-002); this feature never writes to the connection itself |
| Client Contact | FEAT-25.SPEC-006, FEAT-25.SPEC-007 | Establishes Owen's identity for the Own-only view entitlement and the notification recipient |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia marks a Paid invoice Refunded with the full paid amount | Validate the amount equals the amount paid, set Invoice status Refunded, record the refunded amount against the Payment, preserve the original Paid record | Standalone Automation | SPEC-003 |
| Nadia marks a Paid invoice Refunded with less than the full paid amount | Validate the amount does not exceed the amount paid, set Invoice status Partially refunded, record the partial amount against the Payment | Standalone Automation | SPEC-003 |
| Nadia attempts to enter a refund amount greater than the amount paid | Refuse the entry and show the amount actually paid | Inline in SPEC-001 (form validation), governed by SPEC-006 | SPEC-001 / SPEC-006 |
| Nadia marks a project Cancelled | Set the project's stage to Cancelled and stamp `cancelled_at`; leave every other project record (proposal, milestones, deliverables, invoices, activity trail) unchanged | Standalone Automation | SPEC-004 |
| Nadia attempts to mark a project Cancelled that changed state since it was loaded (a second session raced her) | Reject the action and show the project's refreshed current state | Inline in SPEC-002 (reject-with-refresh outcome), governed by SPEC-006 | SPEC-002 / SPEC-006 |
| A refund or cancellation is recorded | Write an append-only Activity Log Entry (XBR-05) | Cross-feature — owned by Immutable Activity & Audit Trail (FEAT-13) | FEAT-13 responsibility |
| A refund is recorded, or a project is cancelled | Email Owen that the invoice was refunded/partially refunded or the project cancelled | Standalone Notification | SPEC-007 |
| The payment-processing capability reports a chargeback or reversal on a Paid invoice | Apply the inbound event: set the Invoice Disputed alongside its preserved Paid record and the Payment Reversed | Standalone Automation, consuming an inbound event owned by FEAT-32's integration spec | SPEC-005 |
| A payment reversal is recorded | Email Nadia immediately so she can respond in her own processor account | Standalone Notification | SPEC-008 |
| Nadia attempts to mark a Refunded invoice Paid again | Refuse the change; a refunded invoice can only return to Paid through a new, logged correction, never a silent status flip | Inline in SPEC-001/SPEC-006 (rule enforcement), the correction itself belongs to FEAT-09/FEAT-10's credit-note or new-invoice flow | SPEC-006 |
| A refund status update or cancellation submission fails to persist (network/server error) | Retry without leaving the invoice or project in an ambiguous status; the prior confirmed status remains authoritative until the retry succeeds | Inline in SPEC-001 and SPEC-002 (Error state) | SPEC-001 / SPEC-002 |
| A refund, cancellation, or reversal changes Invoice/Payment totals | Reflect the change in dashboard and export totals | Cross-feature — owned by Freelancer Financial Dashboard (FEAT-12) and Accounting Export (FEAT-22) | FEAT-12 / FEAT-22 responsibility |
| Dana opens a read-only support session while a refund/cancellation exists on the account | Show the resulting Refunded/Cancelled/Disputed status, never an action to change it | Cross-feature — owned by Operator Support Access (FEAT-31) | FEAT-31 responsibility |

## Shared Context

**Shared Entities:**
- Invoice -- read by SPEC-001 for display and the paid-amount limit; updated by SPEC-003 (Refunded, Partially refunded) and SPEC-005 (Disputed, preserving Paid). Fields touched: `status`.
- Project -- read by SPEC-002 for display; updated by SPEC-004 (`stage` → Cancelled, `cancelled_at`).
- Payment -- read by SPEC-003 and SPEC-005 to establish the amount paid and confirm a genuine reversal; updated by SPEC-003 (refunded amount) and SPEC-005 (status Reversed).
- Payment Account Connection -- read by SPEC-005 only, as the source of the inbound reversal/chargeback notice; never written by this feature (owned by FEAT-32).
- Client Contact -- read by SPEC-006 and SPEC-007 to establish Owen's Own-only view entitlement and notification recipient.

**Shared UI Patterns:**
- Status-alongside-record display -- SPEC-001 and SPEC-002 both show the new status (Refunded/Partially refunded/Disputed, Cancelled) next to the original Paid or active record rather than replacing it, so nothing already recorded ever appears to disappear, matching the Data Notes field's "never overwriting it."
- Optional reason capture -- SPEC-001 and SPEC-002 both offer the same freelancer-entered, optional reason field, stored with the timestamp of the transition.

**Shared Validation:**
- SPEC-006 defines the authorization, refund-amount, refunded-cannot-be-repaid, and concurrency rules. SPEC-001 through SPEC-005 all reference SPEC-006 rather than restating role gates or limits.

## Internal Dependency Map

```
SPEC-001 (Mark Invoice Refunded Screen) -> [Nadia submits a refund amount and optional reason] -> SPEC-006 (Authorization and Validation Rules) -> [pass] -> SPEC-003 (Refund & Partial Refund Recording)
SPEC-003 (Refund & Partial Refund Recording) -> [invoice set to Refunded/Partially refunded] -> SPEC-001 (Mark Invoice Refunded Screen reflects the new status)
SPEC-003 (Refund & Partial Refund Recording) -> [refund recorded] -> SPEC-007 (Refund & Cancellation Notification)
SPEC-002 (Mark Project Cancelled Screen) -> [Nadia submits a cancellation and optional reason] -> SPEC-006 (Authorization and Validation Rules) -> [pass] -> SPEC-004 (Project Cancellation Recording)
SPEC-004 (Project Cancellation Recording) -> [project set to Cancelled] -> SPEC-002 (Mark Project Cancelled Screen reflects the new status)
SPEC-004 (Project Cancellation Recording) -> [cancellation recorded] -> SPEC-007 (Refund & Cancellation Notification)
SPEC-005 (Payment Reversal (Chargeback) Recording) -> [inbound reversal notice applied] -> SPEC-008 (Payment Reversal Notification)
SPEC-005 (Payment Reversal (Chargeback) Recording) -> [invoice set to Disputed] -> SPEC-001 (Mark Invoice Refunded Screen shows Disputed alongside Paid)
SPEC-001 (Mark Invoice Refunded Screen) -> [checks access and refund limits using] -> SPEC-006 (Authorization and Validation Rules)
SPEC-002 (Mark Project Cancelled Screen) -> [checks access and concurrency using] -> SPEC-006 (Authorization and Validation Rules)
```

**Default Entry:** SPEC-001 (Mark Invoice Refunded Screen) and SPEC-002 (Mark Project Cancelled Screen) are both reached from an existing record's detail view -- the invoice detail owned by FEAT-09, and the project detail owned by FEAT-01 -- never from a dedicated home screen of their own, since this feature only ever acts on a record that already exists.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-25.SPEC-001 | Inbound | FEAT-09 (Invoice Generation & Sending) | Nadia reaches the refund action from the invoice detail view FEAT-09 owns | Nadia opens a Paid invoice she needs to refund |
| FEAT-25.SPEC-003 | Inbound | FEAT-10 (Invoice Payment Processing) | The refund limit and eligibility depend on the Payment record FEAT-10 owns (amount paid, status) | Nadia submits a refund |
| FEAT-25.SPEC-002 | Inbound | FEAT-01 (Client & Project Management) | Nadia reaches the cancellation action from the project detail view FEAT-01 owns | Nadia opens a project she needs to cancel |
| FEAT-25.SPEC-004 | Outbound | FEAT-01 (Client & Project Management) | Cancellation sets the project's derived `stage`, which FEAT-01 computes and displays alongside proposal/milestone/invoice state | A project is marked Cancelled |
| FEAT-25.SPEC-004 | Outbound | FEAT-03 (Proposal Acceptance) | An unaccepted proposal on a cancelled project stays open until Nadia revises/re-sends it or cancels the project (XBR-25) — this feature does not touch the Proposal record itself | A project with an unaccepted proposal is cancelled |
| FEAT-25.SPEC-005 | Inbound | FEAT-32 (Payment Account Connection) | The reversal/chargeback notice is relayed from FEAT-32's own integration spec (FEAT-32.SPEC-002); this feature carries no Integration spec of its own for that capability and only consumes the inbound event (XBR-21) | The payment-processing capability reports a reversal or chargeback |
| FEAT-25.SPEC-005 | Inbound | FEAT-10 (Invoice Payment Processing) | A reversal applies only to an invoice FEAT-10 has already marked Paid; the preserved Paid record and the Payment being reversed are FEAT-10's | The payment-processing capability reports a reversal or chargeback |
| FEAT-25.SPEC-003, FEAT-25.SPEC-004, FEAT-25.SPEC-005 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Every refund, partial refund, cancellation, and reversal writes an append-only trail entry (XBR-05) | A refund, cancellation, or reversal is recorded |
| FEAT-25.SPEC-003, FEAT-25.SPEC-005 | Outbound | FEAT-12 (Freelancer Financial Dashboard) | Dashboard totals derive from Invoice and Payment records, including refunds and reversals (XBR-22) | A refund or reversal is recorded |
| FEAT-25.SPEC-003, FEAT-25.SPEC-005 | Outbound | FEAT-22 (Accounting Export) | Export totals derive from Invoice and Payment records, including refunds and reversals (XBR-22) | A refund or reversal is recorded |
| FEAT-25.SPEC-007, FEAT-25.SPEC-008 | Outbound | FEAT-14 (Notifications & Email) | Both notification emails are sent through the transactional email delivery capability, whose Integration spec is owned by FEAT-14; this feature carries no Integration spec of its own for that capability | A refund, cancellation, or reversal is recorded |
| FEAT-25.SPEC-001, FEAT-25.SPEC-002 | Inbound | FEAT-31 (Operator Support Access) | Dana views the resulting Refunded/Cancelled/Disputed status read-only inside a logged support session FEAT-31 owns; she never reaches this feature's own screens and cannot change anything | Dana opens a support session on this account |

## Non-Functional Notes

**Data volumes / growth:** Refunds and cancellations are expected to be infrequent relative to invoice and project volume ("most projects never need it," product-features.md Rationale), scaling with the same freelancer/client base as the rest of the product (a few thousand freelancers in year one, each with 3–15 active clients, per scope-boundaries.md SC-21) — no distinct growth concern beyond that baseline. This feature emits `invoice_marked_refunded`, `project_marked_cancelled`, `partial_refund_recorded`, and `payment_reversal_recorded` signals (product-features.md, Signals field); SPEC-003 fires the refund signals, SPEC-004 fires the cancellation signal, and SPEC-005 fires the reversal signal.

**Responsiveness:** SPEC-001 and SPEC-002 are freelancer-facing status updates rather than client-facing pages, so no specific 2-second mobile responsiveness target applies (assumptions-constraints.md ASMP-21 names client-facing pages specifically); they still show a clear in-progress state while the update persists rather than allowing a second submission (product-features.md, States field: "a failed status update is retried without leaving the invoice in an ambiguous state").

**Data sensitivity / privacy:** Invoice and Payment remain GDPR-class financial records with personal data (payer identity, billing details) even once refunded or disputed, and no card or bank credentials are ever captured here — that handling belongs entirely to the payment-processing capability (assumptions-constraints.md ASMP-24; scope-boundaries.md SC-10). Every record this feature touches is evidentiary once created and never silently altered — only new, logged transitions are added (assumptions-constraints.md ASMP-25; XBR-04).

**Compliance flags:** Refunded and Disputed Invoice records, and Payment Reversed records, may remain subject to legal financial-record retention even after account deletion (feature-dependency-map.md, Entity: Invoice and Entity: Payment, Data Sensitivity). SPEC-001 and SPEC-002 must remain usable with a screen reader and keyboard and never rely on colour alone to distinguish Refunded/Cancelled/Disputed from the original status (assumptions-constraints.md ASMP-27).

**Offline/Degraded:** Per the feature's States field, refund and cancellation updates require connectivity to persist reliably as part of the record; SPEC-001 and SPEC-002 tell Nadia plainly when the action needs a connection rather than appearing to succeed offline (assumptions-constraints.md ASMP-27).

## Non-Goals

- **Issuing refunds or fighting chargebacks inside Clientroom** -- Excluded per scope-boundaries.md (SC-18): refunds are issued by the freelancer through her own payment-processing account, and dispute responses happen there too; this feature only records the outcome truthfully.
- **The platform holding or moving client funds** -- Excluded per BRIEF.md's Constraints and scope-boundaries.md (SC-10): every payment and refund lands in or leaves the freelancer's own connected processor account, never the product.
- **Partial payments or instalments on a single invoice** -- Excluded per scope-boundaries.md (SC-17): an invoice is paid in full or not at all, which is why this feature's refund limit is expressed against the single full Payment rather than a set of instalments; instalments, where needed, are separate invoices through FEAT-04.
- **Automatic purge of Refunded, Disputed, or Cancelled records** -- Intentional lifecycle decision surfaced by the CRUD matrix: Invoice, Payment, and Project records marked by this feature are retained for the life of the freelancer's account with no automatic purge, removed only on account deletion (FEAT-24) subject to legal financial-record retention (feature-dependency-map.md, Entity: Invoice, Payment, and Project, Data Sensitivity).
- **An in-product project delete** -- Intentional lifecycle decision confirmed by the dependency map: cancellation is the only stop-work action this feature offers; no delete of a Project exists anywhere except account deletion (FEAT-24) (feature-dependency-map.md, Entity: Project, Lifecycle).
- **Team or agency-scoped refund/cancellation permissions** -- Excluded per scope-boundaries.md (SC-01): v1 is solo-freelancer only, so this feature's Access field grants Nadia alone the ability to act, with no bookkeeper or delegated role to model.



# Screen Spec: Mark Invoice Refunded Screen

## Overview

**Name:** Mark Invoice Refunded Screen
**ID:** FEAT-25.SPEC-001
**Type:** Screen
**Purpose:** Nadia marks a paid invoice Refunded or Partially refunded, entering the refunded amount and an optional reason, from the invoice detail view.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Entering a refund against a single refund-eligible invoice: full amount or a partial amount, plus an optional freelancer-entered reason. Refund-eligible means exactly one of two combinations (FEAT-25.SPEC-006, Refund eligibility rule): Invoice `status` Paid with its Payment in Succeeded status (paid through the platform), or Invoice `status` Paid (recorded by freelancer) with its Payment in Recorded manually status (paid off-platform and recorded by Nadia via FEAT-10.SPEC-005). Recording a refund against an off-platform payment is the case the feature's "records that a refund was issued outside the platform" capability fits most directly.
- Showing the invoice's amount paid so Nadia can see the ceiling her refund amount cannot cross
- Submitting the refund for validation and recording (FEAT-25.SPEC-003)
- Reflecting the invoice's resulting status (Refunded, Partially refunded, or Disputed if a reversal was recorded in the meantime) once the invoice detail screen (FEAT-09.SPEC-002) is reopened

**Non-Goals:**
- Issuing the refund itself through a payment-processing capability -- excluded per scope-boundaries.md (SC-18): the platform never holds or moves funds, so this screen only records a refund Nadia has already issued through her own processor account; it initiates no money movement
- Marking a project cancelled -- handled by FEAT-25.SPEC-002 (Mark Project Cancelled Screen); this screen acts on a single invoice only
- Recording a second refund against an invoice already Refunded or Partially refunded -- this feature treats a refunded invoice's status as terminal for this action; a further correction is a new, logged event outside this screen's scope (FEAT-25.SPEC-006, XBR-20), not a repeat submission here
- Validating the refund amount or authorizing who may act -- both governed entirely by FEAT-25.SPEC-006 (Authorization and Validation Rules); this screen only surfaces the outcome
- Applying the refund to the Invoice and Payment records -- owned entirely by FEAT-25.SPEC-003 (Refund & Partial Refund Recording); this screen only submits the entry and displays the result

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Nadia taps "Record a refund" on an invoice whose `status` is Paid (Payment Succeeded) or Paid (recorded by freelancer) (Payment Recorded manually); the control is offered for no other invoice status | Invoice reference, invoice's amount paid and currency |
| FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | Nadia opens the invoice from her reversal notification; the invoice is Disputed by then, so no "Record a refund" entry is offered and this screen is not reached | Invoice reference -- see Edge Cases: a Disputed invoice is never eligible for this screen's action |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Enter and submit a full or partial refund with an optional reason | -- |
| Owen (Client Primary Contact) | No | No | Not reachable from Owen's portal navigation; he sees the resulting Refunded/Partially refunded status on his own invoice through FEAT-09's client-facing invoice view (Own-only, view), never this entry screen |
| Priya (Client Reviewer Contact) | No | No | Not reachable; Priya has no billing visibility at all (per the Access Matrix, Invoicing & Payments is None for Reviewer contacts) |
| Dana (Support Operator) | No | No | Not reachable -- Dana never reaches this screen. Inside a logged support session (FEAT-31) she sees only the resulting Refunded/Partially refunded/Disputed status, read-only, on the invoice detail screen (FEAT-09.SPEC-002), where the "Record a refund" control is not rendered in her session; a direct link to this screen opens FEAT-09.SPEC-002 in her read-only session with no error message |
| Unauthenticated | No | No | Redirected to sign-in; after signing in, Nadia lands on the invoice she was trying to refund if the link carried the reference, otherwise on FEAT-09.SPEC-001 (Invoice List) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- an in-progress amount or reason entry is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Record a Refund" with a back arrow (returns to FEAT-09.SPEC-002, Invoice Detail) and a "Mark Refunded" action button (right-aligned, disabled until the form is valid).

**Body, in order:**
- **Invoice summary (read-only):** Invoice number, client name, amount paid, and currency -- the same values shown on FEAT-09.SPEC-002, so Nadia never has to hold the paid amount in her head while entering a refund.
- **Refund type:** A two-option choice -- "Full amount" (the default) and "Partial amount."
- **Amount:** A currency-formatted number input, in the invoice's currency. Pre-filled with the full amount paid and disabled (read-only display) when "Full amount" is selected; empty and editable, required, when "Partial amount" is selected.
- **Reason (optional):** A multi-line text input for a freelancer-entered note about why the refund was issued.

**Footer:** None -- "Mark Refunded" is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described above, full width; the header action remains reachable at the top of the screen.
- **Medium size class and above:** Same single-column form, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-09.SPEC-002 (Invoice Detail) | Screen closes | Standard navigation transition |
| "Full amount" option | Select | Sets amount to the full paid total; disables direct editing of the amount field | Amount field shows the full paid amount, read-only | Field visibly locks with the full amount displayed |
| "Partial amount" option | Select | Clears the amount field and makes it editable | Amount field becomes empty and editable | Field visibly unlocks; focus moves to it |
| Amount input | Type (Partial amount only) | Captures the entered value | Field shows entered value | Standard input state |
| Amount input | Blur | Triggers validation via FEAT-25.SPEC-006 (amount required, greater than zero, not exceeding the amount paid) | Error state on field if invalid | Exact error message per FEAT-25.SPEC-006 below the field |
| Reason input | Type | Captures the optional note | Field shows entered text | Standard input state |
| "Mark Refunded" button | Tap | 1. Validate the form via FEAT-25.SPEC-006. 2. If valid, trigger FEAT-25.SPEC-003 (Refund & Partial Refund Recording). | Button shows a loading state during submission | Success: toast "Invoice marked {Refunded / Partially refunded}" and navigate to FEAT-09.SPEC-002 showing the new status. Failure: inline error per the Error state below. |
| "Mark Refunded" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Refund type options -> Amount input (when editable) -> Reason input -> Mark Refunded.
- **Validation announcements:** When the amount field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Submission feedback:** The success toast is announced on completion; on validation failure, focus moves to the amount field if it is the failing field.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton layout for the invoice summary and form | Screen opens | The invoice's amount paid and currency finish loading |
| Loaded (default) | Form shown with "Full amount" pre-selected and the amount field showing the full paid total, read-only; Reason empty; "Mark Refunded" enabled | Screen opens on a refund-eligible invoice (Paid with a Succeeded Payment, or Paid (recorded by freelancer) with a Recorded manually Payment) | Nadia changes the refund type, edits the reason, or taps Mark Refunded |
| Editing (partial) | Amount field empty and editable, Reason as entered | Nadia selects "Partial amount" | Nadia re-selects "Full amount", or submits |
| Validation Error | The amount field shows its error message below it; Mark Refunded remains enabled to allow retry | Amount validation fails on blur or submit | Nadia corrects the amount and it re-validates |
| Submitting | "Mark Refunded" button shows a loading spinner; all inputs disabled | Nadia taps Mark Refunded with a valid form | FEAT-25.SPEC-003 completes or fails |
| Error | Error banner at the top of the form: "Couldn't record this refund. Try again." with a Retry button; entered amount and reason are preserved | FEAT-25.SPEC-003 reports a failure | Nadia taps Retry and the submission succeeds |
| Offline/Degraded | Banner "You're offline -- this refund can't be recorded until you reconnect." at the top; the form remains editable but Mark Refunded is disabled | Connectivity lost while this screen is open | Connectivity restored -- Mark Refunded re-enables; nothing is queued, since a refund status change must persist immediately as part of the record (assumptions-constraints.md ASMP-27) |

## Validation Rules

Validation governed by FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules). See that spec for the refund-amount ceiling, the no-partial-payment corollary, and the refunded-cannot-be-repaid-without-correction rule. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-09.SPEC-002 (Invoice Detail) | FEAT-09 |
| Successful "Mark Refunded" submission | FEAT-09.SPEC-002 (Invoice Detail), showing the new status | FEAT-09 |
| Retry after Error state succeeds | FEAT-09.SPEC-002 (Invoice Detail), showing the new status | FEAT-09 |

## Data Model

**Creates:** None -- this screen creates no new record; the refund is applied to the existing Invoice and Payment by FEAT-25.SPEC-003.
**Reads:** Invoice -- invoice number, currency, `status` (must be Paid, or Paid (recorded by freelancer), to reach this screen). Payment -- `status` (must be Succeeded for a Paid invoice, or Recorded manually for a Paid (recorded by freelancer) invoice) and amount (the ceiling shown and validated against).
**Updates:** None directly -- submission hands the entered amount and reason to FEAT-25.SPEC-003, which performs the actual Invoice and Payment updates.
**Deletes:** None.

## Business Rules

- Only a refund-eligible invoice reaches this screen: Invoice `status` Paid with a Payment in Succeeded status, or Invoice `status` Paid (recorded by freelancer) with a Payment in Recorded manually status (FEAT-25.SPEC-006, Refund eligibility rule). FEAT-09.SPEC-002 shows the entry point on those two combinations only; an invoice that is Generated, Sent, Payment pending, Overdue, Refunded, Partially refunded, Disputed, or Corrected offers no path here.
- The screen behaves identically for both eligible combinations; a manually recorded payment is refunded exactly like a platform-processed one, since the screen only records a refund Nadia already issued outside the platform.
- The amount field's ceiling is the invoice's amount paid, per FEAT-25.SPEC-006's refund-amount limit (XBR-20).
- Submission triggers FEAT-25.SPEC-003, which is the sole owner of the Invoice status and Payment record changes; this screen never writes those fields itself.
- A recorded refund cannot be undone from this screen -- reversing course requires a new, logged correction outside this feature's scope (XBR-20), consistent with the product's record-immutability constraint (XBR-04).

## Edge Cases

- **Nadia navigates away with an entered amount or reason unsaved** -- No confirmation dialog is shown; unlike a multi-field creation form, a partially entered refund carries no risk of an accidental duplicate record, since nothing is written until Mark Refunded succeeds. The form simply discards the entry.
- **Nadia taps Mark Refunded twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **The invoice was refunded, partially refunded, or reported Disputed by a reversal in another session while this screen was open** -- Submission is rejected with "This invoice's status changed since you opened this page. Refresh to see the latest state." and a "Refresh" action reloads the invoice's current status; this is the concurrent-edit conflict behavior for this screen's update to the shared Invoice and Payment entities, per the dependency map's Contention notes (reject-with-refresh) and FEAT-25.SPEC-006.
- **A reversal notice (FEAT-25.SPEC-005) is applied to this exact invoice at the same moment Nadia submits a refund** -- Processor-confirmed reversal status is authoritative over a concurrent manual refund entry (dependency map, Invoice Contention: "processor-confirmed payment status is authoritative over a concurrent manual entry"); the refund submission is rejected with the same refresh message above, and the invoice reopens showing Disputed.
- **Network failure during submission** -- The Error state's banner and Retry button appear; the entered amount and reason are preserved, and the invoice's prior confirmed status (Paid, or Paid (recorded by freelancer)) remains authoritative until the retry succeeds.
- **Nadia enters an amount exactly equal to the amount paid while "Partial amount" is selected** -- Validation passes (the boundary is inclusive), and FEAT-25.SPEC-003 records it as a full refund (Refunded), not Partially refunded, since the resulting status reflects the amount, not which radio option was used to reach it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (inbound/outbound) | Entry point via "Record a refund"; destination after submission or cancellation |
| FEAT-25.SPEC-003 (Refund & Partial Refund Recording) | Triggers (outbound) | Mark Refunded submits the entry for validation and recording |
| FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules) | References (inbound) | Validation and authorization rules applied to the amount field and to who may reach this screen |
| FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | References (inbound) | A reversal applied to this invoice takes precedence over a concurrent refund submission |
| FEAT-31 (Operator Support Access) | References (inbound) | Owns Dana's read-only support session, in which she sees the resulting invoice status on FEAT-09.SPEC-002; no navigation from FEAT-31 into this screen exists |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| refund_screen_opened | refund type available (full only / full and partial), payment origin (platform / recorded manually) | Screen loads on a refund-eligible invoice | N/A -- no success-metrics.md metric is connected to this feature; retained so refund entry is observable rather than invisible |
| refund_submitted | refund type (full / partial), amount, currency | Nadia taps Mark Refunded with a valid form | N/A -- no success-metrics.md metric is connected to this feature; the outcome is measured downstream by FEAT-25.SPEC-003's own signals |
| refund_submission_rejected | reason (validation_error / stale_state / network_failure) | Submission fails for any reason | N/A -- no success-metrics.md metric is connected to this feature; retained so refund friction is observable rather than silent |

## Acceptance Criteria

**FEAT-25.SPEC-001-AC-01:** Given Nadia is on the Invoice Detail screen (FEAT-09.SPEC-002) for an invoice with status Paid, when she taps "Record a refund", then she lands on this screen with "Full amount" pre-selected and the amount field showing the full paid total, read-only.

**FEAT-25.SPEC-001-AC-02:** Given Nadia is on this screen with "Full amount" selected, when she taps "Mark Refunded", then FEAT-25.SPEC-003 records the invoice as Refunded and she sees a toast "Invoice marked Refunded" before returning to FEAT-09.SPEC-002.

**FEAT-25.SPEC-001-AC-03:** Given Nadia selects "Partial amount", when the amount field becomes editable, then it starts empty and focus moves to it.

**FEAT-25.SPEC-001-AC-04:** Given Nadia enters an amount greater than the invoice's amount paid, when she blurs the field, then the field shows the exact error defined by FEAT-25.SPEC-006 and Mark Refunded does not proceed.

**FEAT-25.SPEC-001-AC-05:** Given Nadia enters a partial amount equal to the full amount paid, when she taps Mark Refunded, then FEAT-25.SPEC-003 records the invoice as Refunded, not Partially refunded.

**FEAT-25.SPEC-001-AC-06:** Given Nadia enters a partial amount less than the amount paid, when she taps Mark Refunded, then FEAT-25.SPEC-003 records the invoice as Partially refunded and she sees the toast "Invoice marked Partially refunded."

**FEAT-25.SPEC-001-AC-07:** Given Nadia enters an optional reason, when she submits, then the reason is recorded alongside the refund by FEAT-25.SPEC-003.

**FEAT-25.SPEC-001-AC-08:** Given Nadia leaves the reason field empty, when she submits, then the refund is still recorded with no reason stored, since the reason is optional.

**FEAT-25.SPEC-001-AC-09:** Given Nadia taps Mark Refunded and the submission fails due to a network error, then the Error banner "Couldn't record this refund. Try again." appears with her entered amount and reason preserved.

**FEAT-25.SPEC-001-AC-10:** Given Nadia taps Mark Refunded twice rapidly, then the second tap has no effect while the first submission is in progress.

**FEAT-25.SPEC-001-AC-11:** Given the invoice was already marked Refunded, Partially refunded, or Disputed in another session since Nadia opened this screen, when she taps Mark Refunded, then the submission is rejected with "This invoice's status changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-001-AC-12:** Given a payment reversal is applied to this exact invoice by FEAT-25.SPEC-005 at the same moment Nadia submits a refund, then the processor-confirmed reversal wins, the refund submission is rejected with the refresh message, and the invoice reopens showing Disputed.

**FEAT-25.SPEC-001-AC-13:** Given Nadia loses connectivity while this screen is open, when she looks at the screen, then the banner "You're offline -- this refund can't be recorded until you reconnect." appears and Mark Refunded is disabled.

**FEAT-25.SPEC-001-AC-14:** Given Owen (Client Primary Contact) attempts to reach this screen directly, then it is not reachable from his portal navigation and no such control exists there.

**FEAT-25.SPEC-001-AC-15:** Given Dana (Support Operator) is in a logged support session viewing an invoice that Nadia has marked Refunded, Partially refunded, or Disputed, when she opens the invoice detail screen (FEAT-09.SPEC-002), then she sees the resulting status read-only, no "Record a refund" control is rendered, and she has no path to this screen; a direct link to this screen opens FEAT-09.SPEC-002 in her read-only session with no error message.

**FEAT-25.SPEC-001-AC-16:** Given Nadia's session expires while she has an amount and reason entered, when she re-authenticates, then the entered amount and reason are restored on this screen.

**FEAT-25.SPEC-001-AC-17:** Given Nadia taps "Record a refund" on an invoice, when this screen opens, then a skeleton layout appears for the invoice summary and form until the amount paid and currency finish loading.

**FEAT-25.SPEC-001-AC-18:** Given an invoice with status Paid (recorded by freelancer) whose Payment is in Recorded manually status, when Nadia opens it on FEAT-09.SPEC-002, then "Record a refund" is offered, and when she submits a valid full refund, then FEAT-25.SPEC-003 records the invoice as Refunded and she sees the toast "Invoice marked Refunded."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 7 (loading, loaded, editing, validation error, submitting, error, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Mark Project Cancelled Screen

## Overview

**Name:** Mark Project Cancelled Screen
**ID:** FEAT-25.SPEC-002
**Type:** Screen
**Purpose:** Nadia marks a project Cancelled, entering an optional reason, from the project detail view, without deleting any project history.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Entering a project cancellation with an optional freelancer-entered reason
- Explaining plainly, before confirming, that cancellation preserves every existing record (proposal, milestones, deliverables, invoices, activity trail) rather than removing anything
- Submitting the cancellation for recording (FEAT-25.SPEC-004)
- Reflecting the project's resulting Cancelled stage once the project detail screen (FEAT-01.SPEC-005) is reopened

**Non-Goals:**
- Deleting the project or any of its records -- excluded per the dependency map (Entity: Project, Lifecycle): no in-product project delete exists anywhere except account deletion (FEAT-24); this screen offers cancellation only, never removal
- Marking an invoice refunded -- handled by FEAT-25.SPEC-001 (Mark Invoice Refunded Screen); this screen acts on a project as a whole, not an individual invoice
- Reversing a cancellation -- the product defines no "un-cancel" action; a project cancelled in error is a business situation resolved outside this screen (the freelancer contacting the client directly), consistent with the feature's focus on recording outcomes truthfully rather than editing history
- Validating who may act or the concurrency behavior -- both governed entirely by FEAT-25.SPEC-006 (Authorization and Validation Rules); this screen only surfaces the outcome
- Applying the cancellation to the Project record -- owned entirely by FEAT-25.SPEC-004 (Project Cancellation Recording); this screen only submits the entry and displays the result

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-005 (Project Detail) | Nadia selects "Cancel Project" from the project detail's overflow menu; the entry appears only while the project's derived stage is none of Cancelled, Archived, or Complete | Project reference |

**Cross-feature note:** FEAT-01.SPEC-005's overflow menu, as currently specified, lists Archive and Mark Complete; it lists FEAT-25 only as a "References (inbound)" connection ("Owns the Cancelled transition this screen only displays") with no corresponding outbound navigation entry yet. This screen is written per the Brief's Internal Dependency Map, which names the project detail owned by FEAT-01 as this screen's default entry (feature-overview.md, Default Entry). The missing "Cancel Project" navigation entry on FEAT-01.SPEC-005's side is flagged here as a bidirectional-reference gap for the Cross-Reference Reconciler (Pass D) to resolve when FEAT-01's specs are next revised; it is outside this spec's authority to edit another feature's file.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Enter and submit a cancellation with an optional reason | -- |
| Owen (Client Primary Contact) | No | No | Not reachable from Owen's portal navigation; he sees the resulting Cancelled status on his own project through the client portal (Own-only, view), never this entry screen |
| Priya (Client Reviewer Contact) | No | No | Not reachable; the Client & Project Management capability group is None for Reviewer contacts |
| Dana (Support Operator) | No | No | Not reachable -- Dana never reaches this screen. Inside a logged support session (FEAT-31) she sees only the resulting Cancelled status, read-only, on the project detail screen (FEAT-01.SPEC-005), where the "Cancel Project" entry is not rendered in her session; a direct link to this screen opens FEAT-01.SPEC-005 in her read-only session with no error message |
| Unauthenticated | No | No | Redirected to sign-in; after signing in, Nadia lands on the project she was trying to cancel if the link carried the reference, otherwise on FEAT-01.SPEC-003 (Client & Project Roster) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- an in-progress reason entry is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Cancel Project" with a back arrow (returns to FEAT-01.SPEC-005, Project Detail) and a "Confirm Cancellation" action button (right-aligned).

**Body, in order:**
- **Project summary (read-only):** Project name and client name -- the same identity block shown on FEAT-01.SPEC-005, so Nadia can confirm she is cancelling the right project.
- **Preservation notice:** A fixed explanatory line: "Cancelling stops work on this project but keeps its full history -- proposal, milestones, deliverables, invoices, and activity -- exactly as it is. Nothing is deleted."
- **Reason (optional):** A multi-line text input for a freelancer-entered note about why the project is being cancelled.

**Footer:** None -- "Confirm Cancellation" is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described above, full width; the header action remains reachable at the top of the screen.
- **Medium size class and above:** Same single-column form, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) | Screen closes | Standard navigation transition |
| Reason input | Type | Captures the optional note | Field shows entered text | Standard input state |
| "Confirm Cancellation" button | Tap | 1. Show an inline confirmation step (see States: Confirming). 2. On confirming a second time, trigger FEAT-25.SPEC-004 (Project Cancellation Recording). | Button shows a loading state during submission after the second confirmation | Success: toast "Project cancelled" and navigate to FEAT-01.SPEC-005 showing the Cancelled stage badge. Failure: inline error per the Error state below. |
| "Confirm Cancellation" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Reason input -> Confirm Cancellation.
- **Confirmation announcements:** The confirming-step dialog text is announced to assistive technology when it appears; the success toast is announced on completion.
- **Validation announcements:** A stale-state rejection dialog receives focus immediately when it appears.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton layout for the project summary and form | Screen opens | The project's name and client finish loading |
| Loaded (default) | Form shown with Reason empty; Confirm Cancellation enabled | Screen opens on an eligible project | Nadia edits the reason or taps Confirm Cancellation |
| Confirming | A blocking dialog restates the preservation notice and asks "Cancel this project? This cannot be undone from here." with "Cancel Project" and "Keep Editing" options | Nadia taps Confirm Cancellation the first time | Nadia confirms a second time (submits) or dismisses (returns to Loaded) |
| Submitting | "Confirm Cancellation" button shows a loading spinner; the reason input is disabled | Nadia confirms the dialog | FEAT-25.SPEC-004 completes or fails |
| Error | Error banner at the top of the form: "Couldn't cancel this project. Try again." with a Retry button; the entered reason is preserved | FEAT-25.SPEC-004 reports a failure | Nadia taps Retry and the submission succeeds |
| Offline/Degraded | Banner "You're offline -- this cancellation can't be recorded until you reconnect." at the top; the form remains editable but Confirm Cancellation is disabled | Connectivity lost while this screen is open | Connectivity restored -- Confirm Cancellation re-enables; nothing is queued, since a cancellation must persist immediately as part of the record (assumptions-constraints.md ASMP-27) |

## Validation Rules

Validation governed by FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules). See that spec for the reject-with-refresh concurrency behavior applied to this screen's submission. This screen has no field-level format validation beyond the optional reason's presence being unconstrained -- an empty reason is always valid.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-005 (Project Detail) | FEAT-01 |
| Successful "Confirm Cancellation" submission | FEAT-01.SPEC-005 (Project Detail), showing the Cancelled stage | FEAT-01 |
| Retry after Error state succeeds | FEAT-01.SPEC-005 (Project Detail), showing the Cancelled stage | FEAT-01 |
| "Keep Editing" on the confirming dialog | Returns to the Loaded state on this same screen | -- |

## Data Model

**Creates:** None -- this screen creates no new record; the cancellation is applied to the existing Project by FEAT-25.SPEC-004.
**Reads:** Project -- project_name, client (owning client's name), `stage` (must resolve to a state this screen can act on -- see Business Rules).
**Updates:** None directly -- submission hands the entered reason to FEAT-25.SPEC-004, which sets `stage` to Cancelled and stamps `cancelled_at`.
**Deletes:** None.

## Business Rules

- This screen is reachable only for a project whose derived stage is none of Cancelled, Archived, or Complete (FEAT-25.SPEC-006). A project already Cancelled, Archived, or Complete offers no "Cancel Project" entry on FEAT-01.SPEC-005: Complete and Cancelled are mutually exclusive terminal states (FEAT-01.SPEC-011 precedence order), so a project Nadia has marked Complete is not cancelled afterward. Every other derived stage can be cancelled.
- Submission triggers FEAT-25.SPEC-004, which is the sole owner of the Project's `stage` and `cancelled_at` fields; this screen never writes those fields itself.
- Cancellation preserves every existing Proposal, Payment Schedule, Milestone, Deliverable, Invoice, and Activity Log Entry unchanged (XBR-25); this screen's preservation notice states that plainly before Nadia confirms.
- An unaccepted proposal on the project stays open until Nadia separately revises and re-sends it, or the project is cancelled -- cancelling here does not itself change the Proposal record (XBR-25); FEAT-02 owns any further action on it.

## Edge Cases

- **Nadia navigates away with an entered reason unsaved** -- No confirmation dialog beyond the Confirming step's own dialog is shown for simply leaving the reason field; the form discards the entry since nothing is written until Confirm Cancellation succeeds.
- **Nadia taps Confirm Cancellation twice rapidly on the confirming dialog** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **The project's state changed since this screen loaded (a second session marked it Complete, Archived, or already Cancelled)** -- Submission is rejected with "This project's state changed since you opened this page. Refresh to see the latest state." and a "Refresh" action reloads the project's current state; this is the concurrent-edit conflict behavior for this screen's update to the shared Project entity, per the dependency map's Contention note for Project (reject-with-refresh) and FEAT-25.SPEC-006.
- **A milestone approval or proposal acceptance fires for this same project at the same moment Nadia confirms cancellation** -- Nadia's explicit Cancelled transition takes precedence and is never overwritten by a system-driven stage change (dependency map, Project Contention: "System-driven stage changes... never overwrite a freelancer's explicit Complete or Cancelled"); the system-driven event still records normally (e.g., the milestone is still Approved), but the project's stage shows Cancelled.
- **Network failure during submission** -- The Error state's banner and Retry button appear; the entered reason is preserved, and the project's prior stage remains authoritative until the retry succeeds.
- **Nadia cancels a project that has open, unpaid invoices** -- No blocking check exists here (unlike Archive's open-items confirmation on FEAT-01.SPEC-007); cancellation is a record of stopped work, not a financial reconciliation step, so open invoices remain exactly as they were and can still be refunded separately through FEAT-25.SPEC-001 if needed.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-005 (Project Detail) | Navigation (inbound/outbound) | Entry point via "Cancel Project"; destination after submission or cancellation (see Entry Points cross-feature note on the current gap in FEAT-01.SPEC-005's own navigation) |
| FEAT-25.SPEC-004 (Project Cancellation Recording) | Triggers (outbound) | Confirm Cancellation submits the entry for recording |
| FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules) | References (inbound) | Authorization and concurrency rules applied to who may reach this screen and how a stale submission is handled |
| FEAT-01.SPEC-011 (Project Stage Derivation) | References (outbound) | The Cancelled stage this screen produces is read and displayed by that spec's derivation formula |
| FEAT-31 (Operator Support Access) | References (inbound) | Owns Dana's read-only support session, in which she sees the resulting Cancelled status on FEAT-01.SPEC-005; no navigation from FEAT-31 into this screen exists |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| cancel_project_screen_opened | project stage at open | Screen loads on an eligible project | N/A -- no success-metrics.md metric is connected to this feature; retained so cancellation entry is observable rather than invisible |
| cancel_project_submitted | reason provided (yes / no) | Nadia confirms cancellation a second time and submission proceeds | N/A -- no success-metrics.md metric is connected to this feature; the outcome is measured downstream by FEAT-25.SPEC-004's own signals |
| cancel_project_submission_rejected | reason (stale_state / network_failure) | Submission fails for any reason | N/A -- no success-metrics.md metric is connected to this feature; retained so cancellation friction is observable rather than silent |

## Acceptance Criteria

**FEAT-25.SPEC-002-AC-01:** Given Nadia is on the Project Detail screen (FEAT-01.SPEC-005), when she selects "Cancel Project" from the overflow menu, then she lands on this screen with the project's name and client shown and the reason field empty.

**FEAT-25.SPEC-002-AC-02:** Given Nadia is on this screen, when she taps Confirm Cancellation, then a blocking dialog appears restating the preservation notice and asking "Cancel this project? This cannot be undone from here." with "Cancel Project" and "Keep Editing" options.

**FEAT-25.SPEC-002-AC-03:** Given Nadia sees the confirming dialog, when she taps "Keep Editing", then the dialog closes and she returns to the Loaded state with her entered reason intact.

**FEAT-25.SPEC-002-AC-04:** Given Nadia sees the confirming dialog, when she taps "Cancel Project" to confirm, then FEAT-25.SPEC-004 records the project as Cancelled and she sees a toast "Project cancelled" before returning to FEAT-01.SPEC-005 showing the Cancelled stage badge.

**FEAT-25.SPEC-002-AC-05:** Given Nadia enters an optional reason, when she confirms cancellation, then the reason is recorded alongside the cancellation by FEAT-25.SPEC-004.

**FEAT-25.SPEC-002-AC-06:** Given Nadia leaves the reason field empty, when she confirms cancellation, then the cancellation is still recorded with no reason stored, since the reason is optional.

**FEAT-25.SPEC-002-AC-07:** Given Nadia confirms cancellation and the submission fails due to a network error, then the Error banner "Couldn't cancel this project. Try again." appears with her entered reason preserved.

**FEAT-25.SPEC-002-AC-08:** Given Nadia taps Confirm Cancellation twice rapidly on the confirming dialog, then the second tap has no effect while the first submission is in progress.

**FEAT-25.SPEC-002-AC-09:** Given the project's state changed in another session since Nadia opened this screen (it was marked Complete, Archived, or already Cancelled), when she confirms cancellation, then the submission is rejected with "This project's state changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-002-AC-10:** Given a milestone is approved on this same project at the same moment Nadia confirms cancellation, then the milestone approval still records normally and the project's stage shows Cancelled, since Nadia's explicit cancellation is never overwritten by a system-driven stage change.

**FEAT-25.SPEC-002-AC-11:** Given the project has open, unpaid invoices, when Nadia cancels it, then no blocking confirmation about the open invoices appears and the invoices remain unchanged.

**FEAT-25.SPEC-002-AC-12:** Given Nadia loses connectivity while this screen is open, when she looks at the screen, then the banner "You're offline -- this cancellation can't be recorded until you reconnect." appears and Confirm Cancellation is disabled.

**FEAT-25.SPEC-002-AC-13:** Given Owen (Client Primary Contact) attempts to reach this screen directly, then it is not reachable from his portal navigation and no such control exists there.

**FEAT-25.SPEC-002-AC-14:** Given Dana (Support Operator) is in a logged support session viewing a project Nadia has marked Cancelled, when she opens the project detail screen (FEAT-01.SPEC-005), then she sees the Cancelled status read-only, no "Cancel Project" entry is rendered, and she has no path to this screen; a direct link to this screen opens FEAT-01.SPEC-005 in her read-only session with no error message.

**FEAT-25.SPEC-002-AC-15:** Given Nadia's session expires while she has a reason entered, when she re-authenticates, then the entered reason is restored on this screen.

**FEAT-25.SPEC-002-AC-16:** Given Nadia selects "Cancel Project", when this screen opens, then a skeleton layout appears for the project summary and form until the project's name and client finish loading.

**FEAT-25.SPEC-002-AC-17:** Given a project whose derived stage is Complete, Archived, or already Cancelled, when Nadia opens its project detail screen (FEAT-01.SPEC-005), then no "Cancel Project" entry is offered and this screen cannot be reached from that project.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 6 (loading, loaded, confirming, submitting, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Refund & Partial Refund Recording

## Overview

**Name:** Refund & Partial Refund Recording
**ID:** FEAT-25.SPEC-003
**Type:** Automation
**Purpose:** Validates and persists Nadia's refund entry -- full or partial, never exceeding the amount paid -- sets the invoice to Refunded or Partially refunded, and preserves the original Paid record rather than overwriting it.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Re-validating the submitted refund amount against the amount paid at the moment of commit (not merely at form load)
- Setting the Invoice's `status` to Refunded (full amount) or Partially refunded (less than the full amount)
- Recording the refunded amount against the Payment record
- Preserving the Invoice's and Payment's prior paid record as the record that remains visible alongside the new status. The prior paid record is one of two refund-eligible combinations (FEAT-25.SPEC-006, Refund eligibility rule): Invoice `status` Paid with Payment `status` Succeeded, or Invoice `status` Paid (recorded by freelancer) with Payment `status` Recorded manually
- Notifying Owen once the refund is recorded (via FEAT-25.SPEC-007)

**Non-Goals:**
- Issuing the refund itself through a payment-processing capability -- excluded per scope-boundaries.md (SC-18): the freelancer has already issued the refund through her own processor account before reaching this automation; no money movement is initiated here
- Collecting the amount and reason from Nadia -- owned entirely by FEAT-25.SPEC-001 (Mark Invoice Refunded Screen); this automation begins where that screen's submission ends
- Applying a processor-reported reversal or chargeback -- owned entirely by FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording); this automation handles only Nadia's own manually entered refund, never an inbound processor event
- Writing the append-only activity trail entry -- owned entirely by Immutable Activity & Audit Trail (FEAT-13, XBR-05); this automation's role ends at the Invoice and Payment updates and the outbound trigger to FEAT-13

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Refund submitted | FEAT-25.SPEC-001 (Mark Invoice Refunded Screen) | Fires when Nadia taps Mark Refunded on a form that passed FEAT-25.SPEC-006's field-level validation, for an invoice that was refund-eligible when the screen loaded (Paid with a Succeeded Payment, or Paid (recorded by freelancer) with a Recorded manually Payment) | Invoice reference, refunded amount, currency, optional reason |

## Processing Logic

1. Receive the invoice reference, refunded amount, and optional reason from the triggering screen.
2. Re-read the Invoice's current `status` and the Payment's current `status` and `amount` at the moment of commit (not the values the screen loaded with).
3. Confirm the invoice is still refund-eligible, meaning exactly one of these two combinations holds: (a) Invoice `status` is Paid and Payment `status` is Succeeded; (b) Invoice `status` is Paid (recorded by freelancer) and Payment `status` is Recorded manually. If neither holds (the invoice is already Refunded, Partially refunded, or Disputed by an intervening reversal, or the Payment `status` does not match its invoice status), stop and report a stale-state outcome.
4. Confirm the refunded amount is greater than zero and does not exceed the Payment's `amount` (FEAT-25.SPEC-006, XBR-20). If it exceeds the amount paid, stop and report a validation-failure outcome (this should already have been caught by the screen's own field validation; this step re-checks authoritatively at commit).
5. Compare the refunded amount to the Payment's `amount`:
   - If equal, set the Invoice's `status` to Refunded.
   - If less, set the Invoice's `status` to Partially refunded.
6. Record the refunded amount and the optional reason against the Payment record, alongside its existing record -- the Payment's prior paid amount, `paid_at`, and `status` (Succeeded, or Recorded manually) are never overwritten; the Payment `status` is left unchanged by a refund.
7. Persist the Invoice and Payment changes together as one committed outcome.
8. Trigger FEAT-25.SPEC-007 (Refund & Cancellation Notification) to email Owen.
9. Signal FEAT-13 (Immutable Activity & Audit Trail) to write the append-only trail entry (XBR-05).
10. Return the new status to the triggering screen for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Full refund recorded | Refunded amount equals the amount paid | Invoice `status` set to Refunded; Payment's refunded amount recorded; prior paid record preserved (Paid/Succeeded, or Paid (recorded by freelancer)/Recorded manually) | Toast "Invoice marked Refunded"; FEAT-09.SPEC-002 shows Refunded alongside the preserved paid record | FEAT-25.SPEC-001, FEAT-09.SPEC-002, FEAT-25.SPEC-007, FEAT-13 |
| Partial refund recorded | Refunded amount is less than the amount paid, greater than zero | Invoice `status` set to Partially refunded; Payment's refunded amount recorded; prior paid record preserved (Paid/Succeeded, or Paid (recorded by freelancer)/Recorded manually) | Toast "Invoice marked Partially refunded"; FEAT-09.SPEC-002 shows Partially refunded alongside the preserved paid record | FEAT-25.SPEC-001, FEAT-09.SPEC-002, FEAT-25.SPEC-007, FEAT-13 |
| Stale-state rejection | At commit time the invoice is no longer in either refund-eligible combination (Paid + Succeeded, or Paid (recorded by freelancer) + Recorded manually) -- already refunded, partially refunded, or disputed, or the Payment `status` does not match | None -- no write occurs | FEAT-25.SPEC-001 shows "This invoice's status changed since you opened this page. Refresh to see the latest state." | FEAT-25.SPEC-001 |
| Validation failure at commit | The refunded amount exceeds the amount paid, or is zero/negative, when re-checked at commit | None -- no write occurs | FEAT-25.SPEC-001 shows the exact error message defined by FEAT-25.SPEC-006 | FEAT-25.SPEC-001 |
| Automation failure | Persisting the Invoice/Payment changes fails after validation passes (e.g., a transient write failure) | No partial write -- the Invoice and Payment are committed together or not at all | FEAT-25.SPEC-001 shows "Couldn't record this refund. Try again." with a Retry button; the invoice's prior confirmed status (Paid, or Paid (recorded by freelancer)) remains authoritative | FEAT-25.SPEC-001 |

## Data Model

**Reads:** Invoice -- `status` (re-checked at commit). Payment -- `amount`, `status` (re-checked at commit).
**Creates:** None.
**Updates:** Invoice -- `status` (Paid or Paid (recorded by freelancer) -> Refunded or Partially refunded). Payment -- refunded amount and the optional reason are recorded against the record, alongside its existing `amount`, `paid_at`, and `status` (Succeeded, or Recorded manually), which are never overwritten.
**Deletes:** None -- the prior paid record is preserved, never replaced (XBR-04).

## Business Rules

- XBR-20: A refund cannot exceed the amount paid; a refunded invoice cannot later be marked Paid again without a logged correction (that correction path belongs to FEAT-09/FEAT-10, not this automation).
- XBR-04: The original Paid record is never silently altered -- the refund is a new, logged transition displayed alongside it, never a replacement of it.
- XBR-22: The refunded amount feeds Financial Dashboard (FEAT-12) and Accounting Export (FEAT-22) totals once committed.
- Validation at commit is authoritative over validation at form load -- the screen's own field-level check (FEAT-25.SPEC-006) is a first pass for user feedback; this automation's step 3-4 re-check (eligible combination, then amount) is what actually gates the write.
- Refund eligibility (FEAT-25.SPEC-006): an invoice paid off-platform (Invoice `status` Paid (recorded by freelancer), Payment `status` Recorded manually via FEAT-10.SPEC-005) is refundable through this automation exactly like a platform-processed payment (Paid + Succeeded); the refund is recorded, never issued, so no processor is involved. Any other invoice status or Payment status pairing is refused as stale-state.
- A refund submitted against an invoice that a reversal has, in the same moment, set to Disputed is refused: processor-confirmed reversal status is authoritative over a concurrent manual refund entry (dependency map, Invoice Contention).

## Edge Cases

- **Two refund submissions for the same invoice arrive from two open sessions of Nadia's at effectively the same time** -- The first to commit sets the Invoice's `status` away from Paid; the second re-checks at step 3, finds the status no longer Paid, and is rejected with the stale-state outcome. Only one refund is ever recorded per invoice.
- **A trigger fires while a previous run for the same invoice is still in flight** -- FEAT-25.SPEC-001 disables Mark Refunded during submission, so a second run for the same invoice cannot start from the same screen instance; a second session's independent submission is handled by the concurrent-firing case above.
- **The refunded amount is exactly equal to the amount paid, entered through the "Partial amount" option** -- Recorded as Refunded, not Partially refunded -- the resulting status reflects the amount, never which form option produced it.
- **A reversal (FEAT-25.SPEC-005) commits for this invoice a moment before this automation's own commit** -- Step 3 finds the Invoice's status is now Disputed, not Paid, and the refund is rejected with the stale-state outcome; the invoice remains Disputed, never simultaneously Refunded.
- **Persisting the Invoice and Payment changes partially fails (one write succeeds, the other does not)** -- The two updates are committed as a single outcome; if either cannot be persisted, neither is applied, and the Invoice remains Paid (or Paid (recorded by freelancer)) until a successful retry.
- **The invoice is Paid (recorded by freelancer) and its Payment is Recorded manually** -- Eligible; step 3 combination (b) passes and the refund proceeds through steps 4-10 unchanged, with the Payment `status` left as Recorded manually.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-001 (Mark Invoice Refunded Screen) | Triggered by (inbound) | Fires on a validated Mark Refunded submission |
| FEAT-25.SPEC-001 (Mark Invoice Refunded Screen) | Affects (outbound) | Returns the new status or a rejection outcome to the screen |
| FEAT-09.SPEC-002 (Invoice Detail) | Affects (outbound) | Displays the resulting Refunded/Partially refunded status alongside the preserved Paid record |
| FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules) | References (inbound) | Supplies the refund-amount ceiling and the stale-state/reject-with-refresh behavior this automation enforces at commit |
| FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | References (inbound) | A reversal committed for the same invoice takes precedence over this automation's own commit |
| FEAT-25.SPEC-007 (Refund & Cancellation Notification) | Triggers (outbound) | A recorded refund fires Owen's notification email |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | A recorded refund writes the append-only trail entry (XBR-05) |
| FEAT-12 (Freelancer Financial Dashboard) | Affects (outbound) | The refunded amount is reflected in dashboard totals (XBR-22) |
| FEAT-22 (Accounting Export) | Affects (outbound) | The refunded amount is reflected in export totals (XBR-22) |

## Analytics and Success Signals

- **invoice_marked_refunded** (invoice reference, amount, currency) -- N/A -- no success-metrics.md metric is connected to this feature; retained per product-features.md's own Signals field so refund activity is observable rather than invisible.
- **partial_refund_recorded** (invoice reference, refunded amount, amount paid, currency) -- N/A -- no success-metrics.md metric is connected to this feature; retained per product-features.md's own Signals field so partial-refund activity is observable rather than invisible.
- **refund_recording_rejected** (reason: stale_state / validation_failure / automation_failure) -- N/A -- no success-metrics.md metric is connected to this feature; retained so refund friction and concurrency collisions are observable rather than silent.

## Acceptance Criteria

**FEAT-25.SPEC-003-AC-01:** Given Nadia submits a refund equal to the full amount paid on a Paid invoice, when this automation processes it, then the Invoice's status is set to Refunded and the prior Paid record remains visible alongside it.

**FEAT-25.SPEC-003-AC-02:** Given Nadia submits a refund less than the full amount paid, when this automation processes it, then the Invoice's status is set to Partially refunded and the refunded amount is recorded against the Payment.

**FEAT-25.SPEC-003-AC-03:** Given the invoice is no longer in a refund-eligible combination at the moment this automation commits (it changed since the screen was loaded), when the commit-time check runs, then no write occurs and FEAT-25.SPEC-001 shows the refresh message.

**FEAT-25.SPEC-003-AC-04:** Given a refund amount that exceeds the amount paid somehow reaches this automation's commit-time check, when the check runs, then the write is refused and FEAT-25.SPEC-001 shows FEAT-25.SPEC-006's exact error message.

**FEAT-25.SPEC-003-AC-05:** Given a refund is successfully recorded, when the commit completes, then FEAT-25.SPEC-007 is triggered to email Owen and FEAT-13 is signaled to write the trail entry.

**FEAT-25.SPEC-003-AC-06:** Given persisting the Invoice and Payment changes fails after validation passes, when the failure occurs, then FEAT-25.SPEC-001 shows "Couldn't record this refund. Try again." and the Invoice's status remains as it was (Paid, or Paid (recorded by freelancer)).

**FEAT-25.SPEC-003-AC-07:** Given two refund submissions for the same invoice arrive from two sessions at effectively the same time, when the first commits, then the second's commit-time check finds the invoice no longer refund-eligible and is rejected.

**FEAT-25.SPEC-003-AC-08:** Given a reversal is recorded for this exact invoice a moment before this automation's own commit, when the commit-time check runs, then it finds the Invoice already Disputed and rejects the refund.

**FEAT-25.SPEC-003-AC-09:** Given a refund amount exactly equal to the amount paid was entered through the "Partial amount" option, when this automation processes it, then the resulting status is Refunded, not Partially refunded.

**FEAT-25.SPEC-003-AC-10:** Given a refund is recorded, when the Financial Dashboard (FEAT-12) or Accounting Export (FEAT-22) totals are next computed, then they reflect the refunded amount.

**FEAT-25.SPEC-003-AC-11:** Given an optional reason accompanies the refund submission, when this automation commits, then the reason is recorded against the Payment alongside the refunded amount.

**FEAT-25.SPEC-003-AC-12:** Given no reason accompanies the refund submission, when this automation commits, then the refund is still recorded with no reason stored.

**FEAT-25.SPEC-003-AC-13:** Given an invoice with status Paid (recorded by freelancer) whose Payment `status` is Recorded manually, when Nadia submits a refund equal to the amount paid, then the Invoice's status is set to Refunded, the Payment `status` remains Recorded manually, and the prior paid record remains visible alongside the new status.

**FEAT-25.SPEC-003-AC-14:** Given an invoice whose status is Paid but whose Payment `status` is neither Succeeded nor a match for that invoice status (or whose status is Generated, Sent, Payment pending, Overdue, or Corrected), when a refund submission reaches the commit-time check, then no write occurs and FEAT-25.SPEC-001 shows the refresh message.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (full refund, partial refund, stale-state, validation failure, automation failure) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Automation Spec: Project Cancellation Recording

## Overview

**Name:** Project Cancellation Recording
**ID:** FEAT-25.SPEC-004
**Type:** Automation
**Purpose:** Persists Nadia's cancellation of a project, sets `cancelled_at`, derives the Cancelled stage, and preserves every existing proposal, milestone, deliverable, and invoice record unchanged.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Re-validating the project's state against a live re-check at the moment of commit (not the state the screen loaded with)
- Setting the Project's `cancelled_at` timestamp and the optional freelancer-entered reason
- Triggering FEAT-01.SPEC-011 (Project Stage Derivation) so the Cancelled stage takes precedence over any milestone- or invoice-driven "In Progress" condition
- Notifying Owen once the cancellation is recorded (via FEAT-25.SPEC-007)

**Non-Goals:**
- Deleting or archiving any project record -- excluded per the dependency map (Entity: Project, Lifecycle): this automation preserves the Proposal, Payment Schedule, Milestones, Deliverables, Invoices, and Activity Log Entries exactly as they were, and touches none of their fields
- Collecting the reason from Nadia -- owned entirely by FEAT-25.SPEC-002 (Mark Project Cancelled Screen); this automation begins where that screen's submission ends
- Computing the stage label itself -- owned entirely by FEAT-01.SPEC-011 (Project Stage Derivation); this automation only sets `cancelled_at`, which that spec reads as one of its precedence-ordered inputs
- Acting on an unaccepted proposal -- excluded per XBR-25: a proposal left open when its project is cancelled stays exactly as it was; revising or voiding it is Nadia's own separate action through FEAT-02, never triggered by this automation

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Cancellation submitted | FEAT-25.SPEC-002 (Mark Project Cancelled Screen) | Fires when Nadia confirms cancellation a second time on the confirming dialog, on a project whose derived stage is none of Cancelled, Archived, or Complete when the screen loaded | Project reference, optional reason |

## Processing Logic

1. Receive the project reference and optional reason from the triggering screen.
2. Re-read the Project's current derived `stage` at the moment of commit (not the value the screen loaded with), by invoking FEAT-01.SPEC-011.
3. Confirm the current derived stage is none of Cancelled, Archived, or Complete (Complete meaning the Project's `completed_at` is set, per FEAT-01.SPEC-011). If it is any of the three, stop and report a stale-state outcome. Because FEAT-25.SPEC-002 offers cancellation only on projects in none of those stages, reaching one of them here always means the project changed since the screen loaded, so the stale-state message applies; a project in any other stage proceeds to plain success.
4. Set the Project's `cancelled_at` to the current timestamp and store the optional reason.
5. Signal FEAT-01.SPEC-011 to recompute the derived `stage`, which resolves to Cancelled once `cancelled_at` is set (taking precedence over any milestone- or invoice-driven "In Progress" condition, per that spec's precedence order).
6. Leave the project's Proposal, Payment Schedule, Milestones, Deliverables, Invoices, and Activity Log Entries entirely untouched.
7. Trigger FEAT-25.SPEC-007 (Refund & Cancellation Notification) to email Owen.
8. Signal FEAT-13 (Immutable Activity & Audit Trail) to write the append-only trail entry (XBR-05).
9. Return the new stage to the triggering screen for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Cancellation recorded | The project's derived stage at commit is none of Cancelled, Archived, or Complete | `cancelled_at` set; reason stored; stage recomputes to Cancelled | Toast "Project cancelled"; FEAT-01.SPEC-005 shows the Cancelled stage badge | FEAT-25.SPEC-002, FEAT-01.SPEC-005, FEAT-01.SPEC-011, FEAT-25.SPEC-007, FEAT-13 |
| Stale-state rejection | The project's derived stage at commit is Cancelled, Archived, or Complete (changed since the screen loaded) | None -- no write occurs | FEAT-25.SPEC-002 shows "This project's state changed since you opened this page. Refresh to see the latest state." | FEAT-25.SPEC-002 |
| Automation failure | Persisting `cancelled_at` fails after the stale-state check passes (e.g., a transient write failure) | No partial write | FEAT-25.SPEC-002 shows "Couldn't cancel this project. Try again." with a Retry button; the project's prior stage remains authoritative | FEAT-25.SPEC-002 |

## Data Model

**Reads:** Project -- derived `stage` and `completed_at` (re-checked at commit, via FEAT-01.SPEC-011).
**Creates:** None.
**Updates:** Project -- `cancelled_at` (set), reason (stored alongside the cancellation).
**Deletes:** None -- no record on the project or any of its related entities is removed (XBR-25).

## Business Rules

- XBR-25: Marking a project cancelled preserves its full history; an unaccepted proposal stays open until Nadia revises and re-sends it or cancels the project -- this automation is the "cancels the project" half of that rule and never itself acts on the proposal.
- Per the dependency map's Contention note for Project: system-driven stage changes (acceptance, approval) never overwrite a freelancer's explicit Cancelled transition; this automation's commit-time check (step 3) exists to keep that guarantee true even under a race, and once `cancelled_at` is set, FEAT-01.SPEC-011's precedence order keeps Cancelled from being overwritten going forward.
- A Complete project cannot be cancelled: Complete and Cancelled are mutually exclusive terminal states (FEAT-01.SPEC-011 precedence order), and this automation refuses a project whose derived stage is Cancelled, Archived, or Complete with the stale-state outcome (FEAT-25.SPEC-006).
- Validation at commit is authoritative over validation at form load -- the screen's own confirming dialog (FEAT-25.SPEC-002) is a first pass for user confirmation; this automation's step 2-3 re-check is what actually gates the write.
- Cancellation carries no financial reconciliation step of its own -- open, unpaid invoices on a cancelled project are untouched by this automation and are refunded separately, if needed, through FEAT-25.SPEC-003.

## Edge Cases

- **Two cancellation confirmations for the same project arrive from two open sessions of Nadia's at effectively the same time** -- The first to commit sets `cancelled_at`; the second re-checks at step 3, finds the stage already Cancelled, and is rejected with the stale-state outcome. Only one cancellation is ever recorded per project.
- **A trigger fires while a previous run for the same project is still in flight** -- FEAT-25.SPEC-002 disables Confirm Cancellation during submission, so a second run for the same project cannot start from the same screen instance; a second session's independent submission is handled by the concurrent-firing case above.
- **A milestone approval or proposal acceptance commits for this same project at effectively the same moment as this automation's own commit** -- Both events are recorded (the milestone is still marked Approved, the proposal still Accepted); FEAT-01.SPEC-011's precedence order resolves the displayed stage to Cancelled regardless of which event committed first, per the dependency map's Contention note.
- **Nadia marks the project Complete via FEAT-01.SPEC-006 at effectively the same moment this automation commits the cancellation** -- Whichever commits first sets its own field (`completed_at` or `cancelled_at`); the second commit's stale-state check (step 3 for this automation, which tests Complete via `completed_at`; FEAT-01.SPEC-006's own check for the reverse order) rejects the second attempt with its own screen's refresh message, so a project never carries both an explicit Complete and an explicit Cancelled transition from the same race.
- **Persisting `cancelled_at` fails after the stale-state check passes** -- No partial write occurs; the project's stage remains whatever it was before this automation ran, until a successful retry.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-002 (Mark Project Cancelled Screen) | Triggered by (inbound) | Fires on a confirmed cancellation submission |
| FEAT-25.SPEC-002 (Mark Project Cancelled Screen) | Affects (outbound) | Returns the new stage or a rejection outcome to the screen |
| FEAT-01.SPEC-005 (Project Detail) | Affects (outbound) | Displays the resulting Cancelled stage badge |
| FEAT-01.SPEC-011 (Project Stage Derivation) | Triggers (outbound) / References (inbound) | This automation sets `cancelled_at`, which that spec's precedence-ordered formula reads to resolve the displayed stage |
| FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules) | References (inbound) | Supplies the reject-with-refresh concurrency behavior this automation enforces at commit |
| FEAT-25.SPEC-007 (Refund & Cancellation Notification) | Triggers (outbound) | A recorded cancellation fires Owen's notification email |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | A recorded cancellation writes the append-only trail entry (XBR-05) |

## Analytics and Success Signals

- **project_marked_cancelled** (project reference, reason provided: yes / no) -- N/A -- no success-metrics.md metric is connected to this feature; retained per product-features.md's own Signals field so cancellation activity is observable rather than invisible.
- **project_cancellation_rejected** (reason: stale_state / automation_failure) -- N/A -- no success-metrics.md metric is connected to this feature; retained so cancellation friction and concurrency collisions are observable rather than silent.

## Acceptance Criteria

**FEAT-25.SPEC-004-AC-01:** Given Nadia confirms cancellation on a project whose derived stage is none of Cancelled, Archived, or Complete, when this automation processes it, then `cancelled_at` is set and the project's derived stage resolves to Cancelled.

**FEAT-25.SPEC-004-AC-02:** Given a cancellation is recorded, when the Project Detail screen (FEAT-01.SPEC-005) is next opened, then it shows the Cancelled stage badge and every proposal, milestone, deliverable, and invoice record is unchanged.

**FEAT-25.SPEC-004-AC-03:** Given the project's derived stage at commit is Cancelled, Archived, or Complete (changed since the screen loaded), when the commit-time check runs, then no write occurs and FEAT-25.SPEC-002 shows the refresh message.

**FEAT-25.SPEC-004-AC-04:** Given a cancellation is successfully recorded, when the commit completes, then FEAT-25.SPEC-007 is triggered to email Owen and FEAT-13 is signaled to write the trail entry.

**FEAT-25.SPEC-004-AC-05:** Given persisting `cancelled_at` fails after the stale-state check passes, when the failure occurs, then FEAT-25.SPEC-002 shows "Couldn't cancel this project. Try again." and the project's prior stage remains displayed.

**FEAT-25.SPEC-004-AC-06:** Given two cancellation confirmations for the same project arrive from two sessions at effectively the same time, when the first commits, then the second's commit-time check finds the stage already Cancelled and is rejected.

**FEAT-25.SPEC-004-AC-07:** Given a milestone is approved on this same project at effectively the same moment as this automation's commit, when both are processed, then the milestone remains Approved and the project's displayed stage still resolves to Cancelled.

**FEAT-25.SPEC-004-AC-08:** Given Nadia marks the same project Complete via FEAT-01.SPEC-006 at effectively the same moment this automation commits the cancellation, when both attempts race, then only the first to commit succeeds and the second is rejected with its own screen's refresh message; if Complete commits first, this automation's step 3 finds `completed_at` set, writes nothing, and FEAT-25.SPEC-002 shows "This project's state changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-004-AC-09:** Given the project has an unaccepted proposal, when this automation records the cancellation, then the Proposal record itself is untouched -- it stays open exactly as it was (XBR-25).

**FEAT-25.SPEC-004-AC-10:** Given the project has open, unpaid invoices, when this automation records the cancellation, then those invoices are untouched.

**FEAT-25.SPEC-004-AC-11:** Given no reason accompanies the cancellation submission, when this automation commits, then the cancellation is still recorded with no reason stored.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (recorded, stale-state, automation failure) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Payment Reversal (Chargeback) Recording

## Overview

**Name:** Payment Reversal (Chargeback) Recording
**ID:** FEAT-25.SPEC-005
**Type:** Automation
**Purpose:** Applies an inbound reversal or chargeback notice relayed from the payment-processing capability to a Paid invoice, setting it Disputed alongside its preserved Paid record and marking the underlying Payment Reversed.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Consuming the reversal/chargeback notice relayed by FEAT-32.SPEC-002's inbound event, already correlated to the affected Invoice
- Setting the Invoice's `status` to Disputed alongside its preserved Paid record
- Setting the underlying Payment's `status` to Reversed
- Notifying Nadia immediately once the reversal is recorded (via FEAT-25.SPEC-008)

**Non-Goals:**
- Receiving the reversal/chargeback event from the payment-processing capability, or correlating it to the right invoice -- owned entirely by FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting); this automation begins with an already-correlated invoice reference
- Responding to the dispute or issuing any counter-evidence to the payment processor -- excluded per scope-boundaries.md (SC-18): dispute responses happen entirely inside Nadia's own processor account; this automation only records the outcome truthfully
- Applying Nadia's own manually entered refund -- owned entirely by FEAT-25.SPEC-003 (Refund & Partial Refund Recording); this automation handles only a processor-reported reversal, never a freelancer-initiated entry
- Marking the project cancelled as a consequence of a reversal -- a reversal never automatically cancels a project; if Nadia chooses to stop work as a result, that is her own separate action through FEAT-25.SPEC-002

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Reversal or chargeback reported | FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Fires when the payment-processing capability reports a dispute on an invoice already recorded as Paid, at any time after payment | Invoice reference (already correlated by FEAT-32.SPEC-002), report timestamp |

## Processing Logic

1. Receive the correlated invoice reference and report timestamp from FEAT-32.SPEC-002's inbound event.
2. Read the Invoice's current `status` and the underlying Payment's current `status`.
3. Duplicate check, before any eligibility test: if the Invoice's `status` is already Disputed and the Payment's `status` is already Reversed, this notice is a duplicate of one already applied. Stop with the "Duplicate / already-Disputed no-op" outcome: write nothing, send no email (FEAT-25.SPEC-008 is not triggered), signal no trail entry, and emit payment_reversal_duplicate_ignored. Do not continue to step 4.
4. Confirm the Invoice is genuinely in a Paid-family status (Paid, Refunded, or Partially refunded) with a Payment in Succeeded status -- a reversal reported against an invoice with no processor-confirmed payment on record cannot apply and is discarded (see Edge Cases). This includes an invoice whose status is Paid (recorded by freelancer) with a Payment in Recorded manually status: an off-platform payment never passed through the payment-processing capability, so there is no processor charge to reverse. It also includes an invoice that is Disputed but whose Payment is not Reversed (an inconsistent state this automation does not repair).
5. Set the Invoice's `status` to Disputed, preserving its prior Paid (or Refunded/Partially refunded) record as the record that remains visible alongside the new status.
6. Set the underlying Payment's `status` to Reversed.
7. Trigger FEAT-25.SPEC-008 (Payment Reversal Notification) to email Nadia immediately.
8. Signal FEAT-13 (Immutable Activity & Audit Trail) to write the append-only trail entry (XBR-05).
9. Return the new status for display on FEAT-09.SPEC-002 (Invoice Detail).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Reversal recorded | The invoice is in a Paid-family status with a Succeeded Payment | Invoice `status` set to Disputed, preserving the prior Paid/Refunded/Partially-refunded record; Payment `status` set to Reversed | FEAT-09.SPEC-002 shows Disputed alongside the preserved prior record; Nadia is emailed immediately (FEAT-25.SPEC-008) | FEAT-09.SPEC-002, FEAT-25.SPEC-001, FEAT-25.SPEC-008, FEAT-13 |
| Duplicate / already-Disputed no-op | At step 3, the invoice is already Disputed and its Payment is already Reversed (a duplicate or repeated notice, including a concurrent second delivery) | None -- no write occurs; the invoice stays Disputed and the Payment stays Reversed | No feedback and no second email -- FEAT-25.SPEC-008 is not triggered and no trail entry is written, since no product event occurred; the ignored duplicate is recorded only through the payment_reversal_duplicate_ignored signal | -- |
| Notice discarded -- no matching payment | The correlated invoice has no Payment in Succeeded status (e.g., the invoice was never actually paid, was paid off-platform with a Recorded manually Payment, or its Payment record no longer exists) and is not an already-Disputed/Reversed duplicate | None -- no write occurs | No feedback -- there is no freelancer-visible state to update and no user waiting on this outcome; the discard itself is logged internally for diagnostic purposes only, not as an Activity Log Entry (no product event occurred) | -- |
| Automation failure | Persisting the Invoice/Payment changes fails after the Paid-family check passes (e.g., a transient write failure) | No partial write -- the Invoice and Payment are committed together or not at all | No end-user-facing feedback (this is a system-to-system event, not a screen submission); the event is retried by the same retry contract FEAT-32.SPEC-002 applies to its own inbound events | FEAT-32.SPEC-002 |

## Data Model

**Reads:** Invoice -- `status`. Payment -- `status`.
**Creates:** None.
**Updates:** Invoice -- `status` (Paid, Refunded, or Partially refunded -> Disputed). Payment -- `status` (Succeeded -> Reversed).
**Deletes:** None -- the prior Paid/Refunded/Partially-refunded record is preserved, never replaced (XBR-04).

## Business Rules

- XBR-21: When the payment processor reports a chargeback or reversal on a paid invoice, the invoice shows Disputed alongside its original Paid record and the freelancer is notified; refunds and dispute responses happen in her own processor account, never inside Clientroom.
- XBR-04: The original Paid record is never silently altered -- Disputed is a new, logged transition displayed alongside it, never a replacement of it.
- Per the dependency map's Invoice Contention note: processor-confirmed payment status is authoritative over a concurrent manual entry -- a reversal committed by this automation takes precedence over any concurrently submitted manual refund (FEAT-25.SPEC-003) or cancellation-adjacent action on the same invoice.
- This automation applies only to Payments that passed through the payment-processing capability (Payment `status` Succeeded). A manually recorded Payment (Recorded manually, invoice Paid (recorded by freelancer)) is outside its scope: a reversal notice against one is discarded, and Nadia's refund of such an invoice is recorded only through FEAT-25.SPEC-003.
- Idempotency: a repeated notice for an invoice already Disputed with a Payment already Reversed is a no-op (step 3) -- it changes nothing, sends no second email, and writes no second trail entry; FEAT-25.SPEC-008 relies on this so Nadia is emailed exactly once per reversal.
- A reversal can be reported against an invoice already Refunded or Partially refunded by Nadia's own prior action, not only one still plainly Paid -- this automation's Paid-family check (step 4) covers all three, since a processor-side dispute can surface after Nadia has already recorded her own refund.
- XBR-22: The Disputed status and the underlying Payment's Reversed status feed Financial Dashboard (FEAT-12) and Accounting Export (FEAT-22) totals once committed.

## Edge Cases

- **A reversal notice arrives for an invoice with no Payment ever recorded as Succeeded** -- Discarded per the Notice discarded outcome; there is no confirmed payment for a reversal to apply against, so no write occurs and no notification fires.
- **A reversal notice arrives for an invoice paid off-platform (status Paid (recorded by freelancer), Payment Recorded manually)** -- Discarded at step 4 per the Notice discarded outcome, since no processor-confirmed payment exists; no write, no notification.
- **Concurrent trigger firing -- two reversal notices for the same invoice arrive at effectively the same time (e.g., a duplicate delivery from the relaying integration)** -- The first to commit sets the Invoice to Disputed and the Payment to Reversed; the second finds the invoice already Disputed and the Payment already Reversed at step 3 and ends with the Duplicate / already-Disputed no-op outcome (no write, no second notification, no second trail entry).
- **A trigger fires while a previous run for the same invoice is still in flight** -- The second run for the same invoice cannot proceed to write until the first completes; once the first commits, the second's read in step 2 sees the already-Disputed/Reversed state and ends with the Duplicate / already-Disputed no-op outcome at step 3.
- **A reversal notice is delivered again long after it was applied (a later retry or redelivery)** -- Same Duplicate / already-Disputed no-op outcome; the invoice's Disputed status and Reversed payment are unchanged and Nadia is not emailed again.
- **Nadia submits a manual refund (FEAT-25.SPEC-003) for this exact invoice at the same moment this automation's reversal commits** -- Whichever commits first wins; if the reversal commits first, FEAT-25.SPEC-003's own commit-time check finds the invoice already Disputed (not Paid) and rejects the refund submission. If the refund commits first, this automation's step 4 still finds the invoice in a Paid-family status (Refunded or Partially refunded) and proceeds to Disputed, since a processor-confirmed reversal is authoritative over the prior manual entry.
- **The invoice was removed via FEAT-24 account deletion before the reversal notice arrives** -- FEAT-32.SPEC-002 discards the notice at its own correlation step, since it cannot be matched to a record that no longer exists; this automation never receives it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Triggered by (inbound) | The "Reversal or chargeback reported" inbound event fires this automation with an already-correlated invoice reference |
| FEAT-09.SPEC-002 (Invoice Detail) | Affects (outbound) | Displays the resulting Disputed status alongside the preserved prior record |
| FEAT-25.SPEC-001 (Mark Invoice Refunded Screen) | References (outbound) | An invoice this automation sets to Disputed is no longer eligible for FEAT-25.SPEC-001's refund entry |
| FEAT-25.SPEC-003 (Refund & Partial Refund Recording) | References (outbound) | A concurrent manual refund submission for the same invoice is superseded by this automation's reversal, per processor-authoritative-over-manual |
| FEAT-25.SPEC-008 (Payment Reversal Notification) | Triggers (outbound) | A recorded reversal fires Nadia's notification email immediately |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | A recorded reversal writes the append-only trail entry (XBR-05) |
| FEAT-12 (Freelancer Financial Dashboard) | Affects (outbound) | The Disputed status and Reversed payment are reflected in dashboard totals (XBR-22) |
| FEAT-22 (Accounting Export) | Affects (outbound) | The Disputed status and Reversed payment are reflected in export totals (XBR-22) |

## Analytics and Success Signals

- **payment_reversal_recorded** (invoice reference, prior status: paid / refunded / partially_refunded) -- N/A -- no success-metrics.md metric is connected to this feature; retained per product-features.md's own Signals field so reversal activity is observable rather than invisible.
- **payment_reversal_notice_discarded** (reason: no_succeeded_payment) -- N/A -- no success-metrics.md metric is connected to this feature; retained so a discarded notice is diagnostically visible rather than silently dropped.
- **payment_reversal_duplicate_ignored** (invoice reference, report timestamp) -- N/A -- no success-metrics.md metric is connected to this feature; retained so duplicate deliveries from the relaying integration are diagnostically visible while sending no second email.

## Acceptance Criteria

**FEAT-25.SPEC-005-AC-01:** Given a Paid invoice with a Succeeded Payment, when the payment-processing capability reports a reversal, then the Invoice's status is set to Disputed and the prior Paid record remains visible alongside it.

**FEAT-25.SPEC-005-AC-02:** Given a reversal is recorded, when the commit completes, then the underlying Payment's status is set to Reversed.

**FEAT-25.SPEC-005-AC-03:** Given a reversal is recorded, when the commit completes, then FEAT-25.SPEC-008 is triggered to email Nadia immediately and FEAT-13 is signaled to write the trail entry.

**FEAT-25.SPEC-005-AC-04:** Given an invoice already Refunded or Partially refunded by Nadia's own prior action, when the payment-processing capability reports a reversal against it, then the Invoice's status is still set to Disputed, since the Paid-family check covers all three prior statuses.

**FEAT-25.SPEC-005-AC-05:** Given a reversal notice arrives for an invoice with no Payment ever recorded as Succeeded, when this automation checks the Payment's status, then no write occurs and no notification fires.

**FEAT-25.SPEC-005-AC-06:** Given two reversal notices for the same invoice arrive at effectively the same time, when the first commits, then the second finds the invoice already Disputed and the Payment already Reversed at step 3, ends with the "Duplicate / already-Disputed no-op" outcome, and applies no further change and sends no second email.

**FEAT-25.SPEC-005-AC-07:** Given Nadia submits a manual refund for an invoice at the same moment a reversal for that invoice commits, when the reversal commits first, then FEAT-25.SPEC-003 finds the invoice already Disputed and rejects the refund submission.

**FEAT-25.SPEC-005-AC-08:** Given Nadia's manual refund commits first for an invoice a moment before a reversal notice for that same invoice arrives, when this automation processes the reversal, then it still finds the invoice in a Paid-family status (Refunded or Partially refunded) and sets it to Disputed.

**FEAT-25.SPEC-005-AC-09:** Given a reversal is recorded, when the Financial Dashboard (FEAT-12) or Accounting Export (FEAT-22) totals are next computed, then they reflect the Disputed status and the Reversed payment.

**FEAT-25.SPEC-005-AC-10:** Given persisting the Invoice and Payment changes fails after the Paid-family check passes, when the failure occurs, then no partial write is left behind and the event is retried by FEAT-32.SPEC-002's own retry contract.

**FEAT-25.SPEC-005-AC-11:** Given an invoice already Disputed with its Payment already Reversed, when a further reversal notice for that invoice arrives at any later time, then the outcome is "Duplicate / already-Disputed no-op": no write occurs, FEAT-25.SPEC-008 is not triggered (no second email), no trail entry is written, and payment_reversal_duplicate_ignored is emitted, not payment_reversal_notice_discarded.

**FEAT-25.SPEC-005-AC-12:** Given an invoice with status Paid (recorded by freelancer) whose Payment is in Recorded manually status, when a reversal notice for it arrives, then the outcome is "Notice discarded -- no matching payment": no write, no notification, and payment_reversal_notice_discarded is emitted.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (recorded, duplicate no-op, discarded, automation failure) | 4 |
| Business Rules | 7 | 7 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Refund, Cancellation & Reversal Authorization and Validation Rules

## Overview

**Name:** Refund, Cancellation & Reversal Authorization and Validation Rules
**ID:** FEAT-25.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs who may mark a refund or cancellation, the refund-amount and no-partial-payment limits, the refunded-cannot-be-repaid-without-correction rule, reject-with-refresh concurrency, which invoice and project states are eligible, and Dana's view-only status visibility.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling
**Governed Entity:** The refund, cancellation, and reversal fields this feature writes on Invoice, Payment, and Project -- not those entities' full field sets, which are governed by their owning features' own Logic/Rule specs (FEAT-09.SPEC-007/FEAT-09.SPEC-008 for Invoice, FEAT-10.SPEC-006 for Payment, FEAT-01.SPEC-011 for Project's derived stage)

## Scope and Non-Goals

**In Scope:**
- Field validation for the refund amount and the optional reason fields this feature introduces on Payment and Project
- The refund-amount ceiling and the no-partial-payment corollary (XBR-20)
- The refunded-cannot-be-repaid-without-correction rule (XBR-20)
- Authorization for marking a refund, marking a cancellation, and viewing the resulting statuses, across every role in the Access Matrix
- The reject-with-refresh concurrency behavior for this feature's writes to Invoice, Payment, and Project
- The processor-authoritative-over-manual rule governing a reversal versus a concurrent manual refund entry
- Which invoice status and Payment status combinations are refund-eligible, and which project stages are cancellable
- Dana's (Support Operator) view-only visibility of this feature's resulting statuses through the owning detail screens, never through this feature's own screens

**Non-Goals:**
- Full field validation for Invoice fields this feature does not write (invoice number, amount, currency, tax line, due date) -- owned entirely by FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due Date Rules); this spec governs only the `status` transitions this feature performs
- Full field validation for Payment fields this feature does not write (method, paid_at, recorded_by) -- owned entirely by FEAT-10.SPEC-006 (Payment Authorization & Validation Rules); this spec governs only the refund amount, the refunded status contribution, and the reversal's Reversed status
- The Project stage derivation formula itself -- owned entirely by FEAT-01.SPEC-011 (Project Stage Derivation); this spec governs only the authorization and concurrency behavior around setting `cancelled_at`, which that formula reads
- Correcting a refunded invoice back to Paid -- excluded per XBR-20: that correction is a new, logged event owned by FEAT-09's credit-note or new-invoice flow, never a status flip this spec or any part of this feature performs

## Governed Entity

**Entity:** The refund/cancellation/reversal fields on Invoice, Payment, and Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| Invoice.status (Refunded / Partially refunded / Disputed values only) | enum | The three status values this feature writes; all other Invoice.status values are owned by FEAT-09, FEAT-10, and FEAT-11 |
| Payment.status (Reversed value only) | enum | The one status value this feature writes; all other Payment.status values are owned by FEAT-10 |
| Payment.refunded_amount | number, feature-introduced | The amount recorded as refunded against a Payment; not yet reflected in the dependency map's Payment field list, since it is introduced by this feature's Key Capabilities (Record a partial refund) -- tracked here as its authoritative definition pending the map's next synthesis pass |
| Payment.refund_reason | text, feature-introduced, optional | The freelancer-entered note accompanying a refund; introduced by this feature alongside `refunded_amount` |
| Project.cancelled_at | timestamp | Set exclusively by this feature; read by FEAT-01.SPEC-011 to derive the Cancelled stage |
| Project.cancellation_reason | text, feature-introduced, optional | The freelancer-entered note accompanying a cancellation; introduced by this feature |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-25.SPEC-001 | Mark Invoice Refunded Screen | On field blur and form submit (refund amount); authorization on screen entry |
| FEAT-25.SPEC-002 | Mark Project Cancelled Screen | Authorization on screen entry; concurrency check on submit |
| FEAT-25.SPEC-003 | Refund & Partial Refund Recording | Commit-time re-validation of the refund amount and the invoice's status; the refunded-cannot-be-repaid rule's write-side enforcement |
| FEAT-25.SPEC-004 | Project Cancellation Recording | Commit-time re-validation of the project's stage; reject-with-refresh enforcement |
| FEAT-25.SPEC-005 | Payment Reversal (Chargeback) Recording | Processor-authoritative-over-manual enforcement when a reversal and a manual refund race |
| FEAT-09.SPEC-002 | Invoice Detail | Authorization on the "Record a refund" entry point's visibility, per role |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Payment.refunded_amount | Required, greater than zero | Always, when a refund is submitted | On blur and on submit (FEAT-25.SPEC-001); re-checked at commit (FEAT-25.SPEC-003) | "Enter an amount greater than zero." | Yes |
| Payment.refunded_amount | Must not exceed the invoice's amount paid | Always | On blur and on submit (FEAT-25.SPEC-001); re-checked at commit (FEAT-25.SPEC-003) | "This amount is more than what was paid. The amount paid was {amount paid}." | Yes |
| Payment.refunded_amount | Numeric, in the invoice's currency, no more than two decimal places | Always | On blur | "Enter a valid amount in {currency}." | Yes |
| Payment.refund_reason | No validation beyond data type -- free text, optional | Always | -- | -- | No |
| Project.cancellation_reason | No validation beyond data type -- free text, optional | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Refund-amount ceiling (XBR-20) | Payment.refunded_amount, Payment.amount | The refunded amount can never exceed the Payment's full amount, since an invoice is paid in full or not at all (SC-17: no partial payments exist for the refund ceiling to divide against) | "This amount is more than what was paid. The amount paid was {amount paid}." |
| No-partial-payment corollary (SC-17) | Payment.amount, Payment.refunded_amount | Because instalments do not exist on a single invoice, the refund ceiling is always the one full Payment amount, never a sum across multiple payments -- this simplifies the ceiling to a single comparison rather than an aggregation | N/A -- this is a structural consequence of SC-17, not itself a user-facing check |
| Refunded-cannot-be-repaid-without-correction (XBR-20) | Invoice.status | An Invoice already Refunded or Partially refunded can never be set back to Paid by any manual action inside this feature or FEAT-10; the only path back to a paid-looking state is a new, logged correction (a credit note or new invoice) owned by FEAT-09, never a status flip | This is enforced by omission -- no control anywhere in the product offers "mark Paid" on a Refunded or Partially refunded invoice; there is no error message because no such attempt is reachable |
| Refund eligibility (invoice + Payment status combinations) | Invoice.status, Payment.status | A refund may be recorded only when exactly one of these combinations holds: (1) Invoice.status Paid with Payment.status Succeeded (paid through the platform); (2) Invoice.status Paid (recorded by freelancer) with Payment.status Recorded manually (paid off-platform, recorded via FEAT-10.SPEC-005). Every other combination is ineligible: Invoice.status Generated, Sent, Payment pending, Overdue, Refunded, Partially refunded, Disputed, or Corrected, or a Paid/Paid (recorded by freelancer) invoice whose Payment.status is not the one paired above (including Reversed, Failed, Initiated, Pending). Both eligible combinations share the same amount ceiling and outcomes. Off-platform payments are deliberately included because the feature records a refund issued outside the platform; they are excluded from processor reversals (FEAT-25.SPEC-005 applies only to Payment.status Succeeded) | Ineligible at entry: the "Record a refund" control is not offered on FEAT-09.SPEC-002. Ineligible at commit: "This invoice's status changed since you opened this page. Refresh to see the latest state." |
| Project cancellability | Project derived stage, Project.completed_at | A project may be cancelled only while its derived stage is none of Cancelled, Archived, or Complete (Complete meaning Project.completed_at is set). Complete and Cancelled are mutually exclusive terminal states (FEAT-01.SPEC-011 precedence order). The stale-state message applies whenever the stage at commit is any of the three (it changed since the screen loaded); any other stage results in plain success | Ineligible at entry: the "Cancel Project" entry is not offered on FEAT-01.SPEC-005. Ineligible at commit: "This project's state changed since you opened this page. Refresh to see the latest state." |
| Processor-authoritative-over-manual (dependency map, Invoice Contention) | Invoice.status, Payment.status | When a processor-reported reversal and a concurrently submitted manual refund race for the same invoice, the reversal wins regardless of arrival order relative to the refund submission's own commit attempt | "This invoice's status changed since you opened this page. Refresh to see the latest state." (shown to Nadia on the losing manual refund submission) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| Mark an invoice Refunded or Partially refunded | Nadia (Freelancer) | Only on her own invoice, only while it is refund-eligible: status Paid with a Succeeded Payment, or status Paid (recorded by freelancer) with a Recorded manually Payment (Refund eligibility rule) | On any other invoice status the "Record a refund" control is not offered; a submission made after the invoice stopped being eligible is refused with "This invoice's status changed since you opened this page. Refresh to see the latest state." |
| Mark an invoice Refunded or Partially refunded | Owen (Client Primary Contact) | Never | The action is not shown anywhere in his portal navigation |
| Mark an invoice Refunded or Partially refunded | Priya (Client Reviewer Contact) | Never | The action is not shown; Priya has no billing visibility at all (Access Matrix: Invoicing & Payments is None) |
| Mark an invoice Refunded or Partially refunded | Dana (Support Operator) | Never | The "Record a refund" control is not rendered in Dana's support session (FEAT-09.SPEC-002) and she never reaches FEAT-25.SPEC-001; a direct link to that screen opens the invoice detail (FEAT-09.SPEC-002) in her read-only session with no error message |
| Mark a project Cancelled | Nadia (Freelancer) | Only on her own project, only while its derived stage is none of Cancelled, Archived, or Complete (Project cancellability rule) | On a project in one of those three stages the "Cancel Project" entry is not offered; a confirmation made after the project reached one of them is refused with "This project's state changed since you opened this page. Refresh to see the latest state." |
| Mark a project Cancelled | Owen (Client Primary Contact) | Never | The action is not shown anywhere in his portal navigation |
| Mark a project Cancelled | Priya (Client Reviewer Contact) | Never | The action is not shown; the Client & Project Management capability group is None for Reviewer contacts |
| Mark a project Cancelled | Dana (Support Operator) | Never | The "Cancel Project" entry is not rendered in Dana's support session (FEAT-01.SPEC-005) and she never reaches FEAT-25.SPEC-002; a direct link to that screen opens the project detail (FEAT-01.SPEC-005) in her read-only session with no error message |
| View the resulting Refunded/Partially refunded/Disputed status on an invoice | Nadia (Freelancer) | Always, on her own invoices | -- |
| View the resulting Refunded/Partially refunded/Disputed status on an invoice | Owen (Client Primary Contact) | Own-only, on his own company's invoices | -- |
| View the resulting Refunded/Partially refunded/Disputed status on an invoice | Priya (Client Reviewer Contact) | Never | Invoice content is hidden entirely from Reviewer contacts (FEAT-09.SPEC-006); a direct link redirects to her portal home with no error message |
| View the resulting Refunded/Partially refunded/Disputed status on an invoice | Dana (Support Operator) | Always, view only, inside a logged support session (FEAT-31), on the invoice detail screen (FEAT-09.SPEC-002) -- the status only, never an action to change it | -- |
| View the resulting Cancelled status on a project | Nadia (Freelancer) | Always, on her own projects | -- |
| View the resulting Cancelled status on a project | Owen (Client Primary Contact) | Own-only, on his own project, through the client portal | -- |
| View the resulting Cancelled status on a project | Priya (Client Reviewer Contact) | Own-only, on her own project (the same portal view Owen sees, per the Access Matrix's Client Portal Access row) | -- |
| View the resulting Cancelled status on a project | Dana (Support Operator) | Always, view only, inside a logged support session (FEAT-31), on the project detail screen (FEAT-01.SPEC-005) -- the status only, never an action to change it | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Invoice.status | Set to Refunded when Payment.refunded_amount equals Payment.amount; set to Partially refunded when less; set to Disputed on a processor-reported reversal, regardless of the prior Paid-family value | On commit (FEAT-25.SPEC-003, FEAT-25.SPEC-005) | No -- the resulting value is always derived from the amount or the event type, never chosen directly by Nadia |
| Payment.status | Set to Reversed on a processor-reported reversal | On commit (FEAT-25.SPEC-005) | No -- this value is set only by the inbound processor event, never by any manual action |
| Project.cancelled_at | Current timestamp | On commit (FEAT-25.SPEC-004) | No -- always the commit-time timestamp |

## Business Rules

- XBR-04: Every record this feature touches is evidentiary and immutable once written -- a refund, cancellation, or reversal is a new, logged transition displayed alongside the prior record, never a replacement of it.
- XBR-05: Every refund, partial refund, cancellation, and reversal writes an append-only Activity Log Entry (owned by FEAT-13, triggered by FEAT-25.SPEC-003/004/005).
- XBR-20: A refund cannot exceed the amount paid; a refunded invoice cannot later be marked Paid again without a logged correction.
- XBR-21: A processor-reported reversal sets the invoice Disputed alongside its preserved Paid record and notifies Nadia; refunds and dispute responses happen in her own processor account, never inside Clientroom.
- XBR-25: Cancellation preserves a project's full history; an unaccepted proposal stays open until Nadia revises and re-sends it or cancels the project.
- Reject-with-refresh concurrency (dependency map, Invoice and Project Contention notes): any write this feature attempts against a shared Invoice or Project entity that has changed since it was loaded is refused, with the refreshed current state shown -- never a silent overwrite and never a merge.
- XBR-29: Dana's support sessions are read-only in every feature, including this one. Per the Brief and FEAT-09.SPEC-002, Dana sees this feature's resulting statuses (Refunded, Partially refunded, Disputed, Cancelled) only on the owning detail screens (FEAT-09.SPEC-002, FEAT-01.SPEC-005), where no refund or cancellation control is rendered in her session; she never reaches FEAT-25.SPEC-001 or FEAT-25.SPEC-002 and cannot change anything. XBR-29 itself only establishes that sessions are read-only; the never-reaches-these-screens behavior comes from the Brief's Cross-Feature Touchpoints and Side-Effect Inventory.

## Edge Cases

- **A refund amount is entered at exactly the amount paid** -- Passes validation (the boundary is inclusive); FEAT-25.SPEC-003 records it as Refunded, not Partially refunded.
- **A refund amount is entered at one currency unit over the amount paid** -- Fails validation with "This amount is more than what was paid. The amount paid was {amount paid}."; the boundary is exclusive on the high side.
- **Nadia attempts to mark a Refunded invoice Paid again through any control anywhere in the product** -- No such control exists; the refunded-cannot-be-repaid rule is enforced by omission rather than a blocking error, since FEAT-09 and FEAT-10 never render a "mark Paid" action on an invoice already in a Refunded-family status.
- **A reversal and a manual refund submission race for the same invoice** -- Whichever commits first wins; if the reversal commits first, the refund submission's own commit-time check (FEAT-25.SPEC-003) finds the invoice already Disputed and is refused with the refresh message. If the refund commits first, the reversal (FEAT-25.SPEC-005) still applies on top of it, since a processor-confirmed reversal is authoritative over the prior manual entry.
- **Dana opens a read-only support session on an account with a Disputed invoice or a Cancelled project** -- She sees the resulting status on the owning detail screen exactly as Nadia would, but no refund or cancellation control is rendered and she never sees an action to change it, per the Support Operator's unconditional view-only entitlement.
- **A project's stage changes (Complete, Archived, or a second Cancelled attempt) between when FEAT-25.SPEC-002 loads and when Nadia confirms cancellation** -- The commit-time check in FEAT-25.SPEC-004 rejects the stale attempt with the refresh message; the project's actual current state is never silently overwritten.
- **Nadia looks for "Cancel Project" on a project she has marked Complete** -- The entry is not offered; a Complete project is never cancelled afterward (Project cancellability rule). No error appears because the entry itself is absent.
- **A refund is attempted on an invoice paid off-platform (Paid (recorded by freelancer), Recorded manually)** -- Eligible; it proceeds exactly like a platform-paid invoice. A processor reversal notice for that same invoice is discarded by FEAT-25.SPEC-005, since no processor charge exists.

## Acceptance Criteria

**FEAT-25.SPEC-006-AC-01:** Given Nadia enters a refund amount of exactly the amount paid, when she submits, then validation passes and FEAT-25.SPEC-003 records the invoice as Refunded.

**FEAT-25.SPEC-006-AC-02:** Given Nadia enters a refund amount one unit over the amount paid, when she blurs the field, then she sees "This amount is more than what was paid. The amount paid was {amount paid}." and the field remains in an error state.

**FEAT-25.SPEC-006-AC-03:** Given Nadia enters a refund amount of zero, when she blurs the field, then she sees "Enter an amount greater than zero."

**FEAT-25.SPEC-006-AC-04:** Given Nadia enters a refund amount with more than two decimal places, when she blurs the field, then she sees "Enter a valid amount in {currency}."

**FEAT-25.SPEC-006-AC-05:** Given Nadia (Freelancer) opens her own invoice with status Paid (Payment Succeeded), or with status Paid (recorded by freelancer) (Payment Recorded manually), when she looks for "Record a refund", then it is shown and reachable in both cases.

**FEAT-25.SPEC-006-AC-06:** Given Owen (Client Primary Contact) looks anywhere in his portal navigation, when he searches for a refund or cancellation action, then none exists.

**FEAT-25.SPEC-006-AC-07:** Given Priya (Client Reviewer Contact) attempts to reach an invoice directly, then invoice content is hidden entirely and she is redirected to her portal home with no error message.

**FEAT-25.SPEC-006-AC-08:** Given Dana (Support Operator) opens a logged support session on an account with a Paid invoice, when she views the invoice detail screen (FEAT-09.SPEC-002), then no "Record a refund" control is rendered, she has no path to FEAT-25.SPEC-001, and a direct link to it opens the invoice detail read-only with no error message.

**FEAT-25.SPEC-006-AC-09:** Given Nadia (Freelancer) opens a project whose derived stage is none of Cancelled, Archived, or Complete, when she looks for "Cancel Project", then it is shown and reachable.

**FEAT-25.SPEC-006-AC-10:** Given Dana (Support Operator) opens a logged support session on an account with an active project, when she views the project detail screen (FEAT-01.SPEC-005), then no "Cancel Project" entry is rendered, she has no path to FEAT-25.SPEC-002, and she sees the project's status read-only.

**FEAT-25.SPEC-006-AC-11:** Given Owen (Client Primary Contact) views his own company's invoice, when it is Refunded or Partially refunded, then he sees the resulting status alongside the preserved Paid record.

**FEAT-25.SPEC-006-AC-12:** Given Priya (Client Reviewer Contact) views her own project's portal page, when the project is Cancelled, then she sees the resulting Cancelled status (per the Access Matrix's Own-only portal view), with no billing content mixed into that view.

**FEAT-25.SPEC-006-AC-13:** Given no control anywhere in the product offers "mark Paid" on a Refunded invoice, when Nadia looks for one, then none exists, and any correction must go through a new credit note or invoice via FEAT-09.

**FEAT-25.SPEC-006-AC-14:** Given a reversal commits first for an invoice, when Nadia's concurrent manual refund submission for that same invoice reaches commit, then it is refused with "This invoice's status changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-006-AC-15:** Given Nadia's manual refund commits first for an invoice, when a reversal notice for that same invoice is then processed, then the reversal still applies and sets the invoice Disputed, since a processor-confirmed reversal is authoritative over the prior manual entry.

**FEAT-25.SPEC-006-AC-16:** Given a project's stage changed (to Complete, Archived, or already Cancelled) between when Nadia loaded the Mark Project Cancelled screen and when she confirms, when she confirms, then the submission is rejected with "This project's state changed since you opened this page. Refresh to see the latest state." and no overwrite occurs; and given the project's stage is any other stage at confirmation, then the cancellation succeeds with the plain "Project cancelled" toast and no stale-state message.

**FEAT-25.SPEC-006-AC-17:** Given an invoice's status changed since Nadia loaded the Mark Invoice Refunded screen, when she submits her refund, then the submission is rejected with the refresh message and no overwrite occurs.

**FEAT-25.SPEC-006-AC-18:** Given an invoice has already reached its one full Payment (SC-17: no instalments exist), when Nadia enters a refund amount, then the ceiling checked is that single Payment's full amount, never a sum across multiple payments.

**FEAT-25.SPEC-006-AC-19:** Given Nadia leaves the refund reason or cancellation reason field empty, when she submits, then no validation error appears, since both reason fields are optional.

**FEAT-25.SPEC-006-AC-20:** Given a reversal is recorded on a previously Refunded or Partially refunded invoice, when the reversal is processed, then it still applies and sets the invoice Disputed, since the Paid-family check covers all three statuses that can precede a reversal.

**FEAT-25.SPEC-006-AC-21:** Given an invoice with status Paid whose Payment is in Succeeded status, or status Paid (recorded by freelancer) whose Payment is in Recorded manually status, when Nadia submits a valid refund amount, then the eligibility check passes and FEAT-25.SPEC-003 records the refund.

**FEAT-25.SPEC-006-AC-22:** Given an invoice whose status is Generated, Sent, Payment pending, Overdue, Refunded, Partially refunded, Disputed, or Corrected, or a Paid invoice whose Payment is Reversed, when Nadia looks for "Record a refund" or a stale submission reaches commit, then the control is not offered, or the submission is refused with "This invoice's status changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-006-AC-23:** Given a project Nadia has marked Complete, when she opens its project detail screen, then no "Cancel Project" entry is offered, and a stale cancellation confirmation submitted for it is refused with the refresh message.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 6 | 6 |
| Authorization Rules | 16 | 16 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |



# Notification Spec: Refund & Cancellation Notification

## Overview

**Name:** Refund & Cancellation Notification
**ID:** FEAT-25.SPEC-007
**Type:** Notification
**Purpose:** Emails Owen when an invoice he was billed is marked Refunded/Partially refunded or when his project is marked Cancelled, so his own record of the relationship stays accurate without asking Nadia.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- The email delivered to Owen when Nadia marks one of his invoices Refunded or Partially refunded
- The email delivered to Owen when Nadia marks his project Cancelled
- Delivery, deduplication, retry, and expiry behavior for both trigger variants

**Non-Goals:**
- Notifying Owen of a payment reversal/chargeback -- this product records reversals as a fact for Nadia's own attention (she is the one who must respond in her own processor account); product-features.md's Communications field names no client-facing reversal email, so none exists here
- Notifying Nadia of her own refund or cancellation action -- she performed the action herself and sees its result immediately on FEAT-25.SPEC-001/FEAT-25.SPEC-002's own success toast; a separate email to the person who just took the action would be redundant
- Notifying Priya -- excluded per the Access Matrix: Priya has no billing visibility at all, and product-features.md's Communications field names Owen as the sole recipient
- Any in-app channel -- product-features.md's Communications field names only "Notification email to Owen"; this product's in-app notification feed (FEAT-29) is a Later-phase feature not yet built, so email is the only channel this spec defines

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on both trigger variants | Owen's sessions are short and triggered by a specific email (BRIEF.md, Target Users & Roles); he is not routinely browsing the portal, so an in-portal-only status change would go unseen until his next unrelated visit |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Refund or partial refund recorded | FEAT-25.SPEC-003 (Refund & Partial Refund Recording) | Fires when a refund commit succeeds | Invoice reference, invoice number, refund type (full / partial), refunded amount, currency, client name, project name |
| Project cancellation recorded | FEAT-25.SPEC-004 (Project Cancellation Recording) | Fires when a cancellation commit succeeds | Project reference, project name, client name |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) -- the Access Matrix entitles him to Own-only view of his own company's invoices and projects; he is the party billed on the refunded invoice or the primary contact for the cancelled project. Owen only, never Priya (no billing visibility per the Access Matrix) and never Nadia (she is the notification's actor, not its recipient).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional record email per XBR-30 ("transactional emails core to the record... always send") | -- | Always on, no opt-out | N/A -- no preference surface exists for this notification; it is not shown as a toggleable row on any Notification Preferences screen, consistent with the transactional treatment product-features.md applies to record-status emails |

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); this notification is transactional and sends immediately regardless of the time of day, consistent with the refund or cancellation it reports being a permanent, timestamped record the moment it is confirmed.

## Content Definition

**Email (refund variant -- full refund):**
- **Subject:** Invoice {invoice_number} has been refunded
- **Body:**
  Hi {owen_first_name},

  Your invoice {invoice_number} for {project_name} has been refunded in full ({refunded_amount} {currency}).

  The original payment record remains on file alongside this update.
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for the affected invoice

**Email (refund variant -- partial refund):**
- **Subject:** Invoice {invoice_number} has been partially refunded
- **Body:**
  Hi {owen_first_name},

  {refunded_amount} {currency} has been refunded against your invoice {invoice_number} for {project_name}.

  The original payment record remains on file alongside this update.
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for the affected invoice

**Email (cancellation variant):**
- **Subject:** {project_name} has been marked cancelled
- **Body:**
  Hi {owen_first_name},

  {freelancer_first_name} has marked {project_name} as cancelled.

  Everything already recorded for this project -- your proposal, approvals, deliverables, and invoices -- remains exactly as it was and stays available to you.
- **CTA (button):** View project -- deep-links to the client-facing project view Owen's own portal provides (FEAT-05, Own-only)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {owen_first_name} | Client Contact -- name | Owen | Greeting renders as "Hi," |
| {invoice_number} | Invoice -- invoice_number | INV-0042 | Never empty -- invoice_number is required at generation (FEAT-09.SPEC-007) |
| {project_name} | Project -- project_name | Brand Refresh | Never empty -- project_name is required at project creation (FEAT-01.SPEC-002) |
| {refunded_amount} | Payment -- refunded_amount | 450.00 | Never empty -- this email only fires once a refund amount has been validated and committed (FEAT-25.SPEC-003) |
| {currency} | Invoice -- currency | USD | Never empty -- currency is required before a project's first invoice (XBR-17) |
| {freelancer_first_name} | Freelancer Account -- name | Nadia | Renders as "Your freelancer" when unavailable, though this field is required on every Freelancer Account and so is never actually empty in practice |

## Delivery Rules

**Batching:** None -- each refund or cancellation event is delivered as its own, individual email the moment it is recorded; the two trigger variants are never combined into one message even if both occur for the same project in quick succession, since a refund and a cancellation are distinct facts about distinct records (an invoice and a project) that Owen may need to act on separately.
**Deduplication:** At most one email per refund commit and one email per cancellation commit -- each is tied to a single, one-time state transition (FEAT-25.SPEC-003, FEAT-25.SPEC-004) that can happen at most once per invoice (Refunded/Partially refunded is not re-entered once set) or once per project (Cancelled is a terminal, non-repeatable transition).
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the refunded or cancelled status itself remains fully visible to Owen the next time he opens the affected invoice or project in his portal, regardless of this email's delivery outcome.
**Expiry:** This notification never expires undelivered in the sense of becoming irrelevant -- a refund or cancellation is a permanent record, so a delayed delivery still carries an accurate message whenever it eventually lands; retries continue for the full retry window above, and after that the delivery-failure warning (not a silently dropped message) is the surviving signal.

## Edge Cases

- **The invoice is later reported Disputed by a reversal after this refund email was already sent** -- No second email is sent by this spec for the reversal; that event is Nadia's own notification (FEAT-25.SPEC-008), never Owen's, per this spec's Non-Goals.
- **Owen's email address bounces on the first delivery attempt** -- Retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`; after the final failure, Nadia sees a delivery warning on the affected project and Owen still sees the refunded or cancelled status directly in his portal.
- **Owen's Client Contact record is removed (erasure request) before this email is delivered** -- Per XBR-27, an erasure request ends access and removes contact details immediately; this notification is cancelled silently rather than delivered to a now-invalid address, since there is no longer a contact entitled to receive it.
- **Two refunds are recorded on two different invoices for the same project at effectively the same time** -- Each produces its own, separate email; they are never merged into one message, since Delivery Rules define no batching for this notification.
- **The project is cancelled and, moments later, one of its invoices is also refunded** -- Two separate emails are sent, one per trigger variant, in the order the two commits actually occurred.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-003 (Refund & Partial Refund Recording) | Triggered by (inbound) | A successful refund commit fires the refund variant of this notification |
| FEAT-25.SPEC-004 (Project Cancellation Recording) | Triggered by (inbound) | A successful cancellation commit fires the cancellation variant of this notification |
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (outbound) | The refund variant's CTA deep-links here |
| FEAT-05 (Client Portal Access) | Navigation (outbound) | The cancellation variant's CTA deep-links to Owen's own client-facing project view |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (inbound) | Delivers this notification and reports delivery, bounce, and failure status |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | Triggers (outbound) | A final delivery failure surfaces as a warning to Nadia on the affected project |

## Analytics and Success Signals

- **refund_cancellation_notification_delivered** (variant: full_refund / partial_refund / cancellation) -- N/A -- no success-metrics.md metric is connected to this feature; retained so delivery of this record-affecting email is observable rather than invisible.
- **refund_cancellation_notification_delivery_failed** (variant, final_failure: yes / no) -- N/A -- no success-metrics.md metric is connected to this feature; retained so silent delivery loss is observable rather than invisible, consistent with the general Notification Delivery Reliability goal this product tracks across features (success-metrics.md, "Notification Delivery Reliability" -- that metric's Connected Feature is Notifications (Email), not this feature, so it is not cited here as this feature's own signal).

## Acceptance Criteria

**FEAT-25.SPEC-007-AC-01:** Given Nadia records a full refund on Owen's invoice, when the commit succeeds, then Owen receives an email with subject "Invoice {invoice_number} has been refunded" and a "View invoice" CTA.

**FEAT-25.SPEC-007-AC-02:** Given Nadia records a partial refund on Owen's invoice, when the commit succeeds, then Owen receives an email with subject "Invoice {invoice_number} has been partially refunded" naming the refunded amount.

**FEAT-25.SPEC-007-AC-03:** Given Nadia marks Owen's project Cancelled, when the commit succeeds, then Owen receives an email with subject "{project_name} has been marked cancelled" and a "View project" CTA to his own portal view.

**FEAT-25.SPEC-007-AC-04:** Given Owen taps "View invoice" on a refund email, then he lands on FEAT-09.SPEC-002 showing the invoice's Refunded or Partially refunded status.

**FEAT-25.SPEC-007-AC-05:** Given Owen taps "View project" on a cancellation email, then he lands on his own client-facing project view showing the Cancelled status.

**FEAT-25.SPEC-007-AC-06:** Given Nadia performs the refund or cancellation herself, then she receives no email from this spec -- only Owen is a recipient.

**FEAT-25.SPEC-007-AC-07:** Given Priya is a contact on the same client company, when a refund or cancellation is recorded, then she receives no email from this spec.

**FEAT-25.SPEC-007-AC-08:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the project for Nadia.

**FEAT-25.SPEC-007-AC-09:** Given a delivery to Owen fails permanently after retries are exhausted, then Nadia sees a delivery warning on the affected project, and Owen still sees the refunded or cancelled status directly in his portal.

**FEAT-25.SPEC-007-AC-10:** Given Owen's Client Contact record is removed by an erasure request before this email is delivered, when the delivery would otherwise fire, then it is cancelled silently and no email is sent to the removed address.

**FEAT-25.SPEC-007-AC-11:** Given two refunds are recorded on two different invoices for the same project at effectively the same time, then Owen receives two separate emails, never one combined message.

**FEAT-25.SPEC-007-AC-12:** Given a project is cancelled and one of its invoices is separately refunded moments later, then Owen receives two separate emails in the order the two events occurred.

**FEAT-25.SPEC-007-AC-13:** Given this notification is transactional, when Owen looks for a way to turn it off, then no preference control for it exists anywhere.

**FEAT-25.SPEC-007-AC-14:** Given a refund or cancellation is recorded at any hour, when the email is ready to send, then it sends immediately with no quiet-hours hold, since this notification is transactional.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 (refund/partial refund, cancellation) | 2 |
| Preference States | 1 (always on, no opt-out) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Payment Reversal Notification

## Overview

**Name:** Payment Reversal Notification
**ID:** FEAT-25.SPEC-008
**Type:** Notification
**Purpose:** Emails Nadia the moment a payment reversal or chargeback is recorded, so she knows to respond in her own processor account.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- The email delivered to Nadia the moment a reversal or chargeback is recorded against one of her invoices
- Delivery, deduplication, retry, and expiry behavior for this single trigger

**Non-Goals:**
- Notifying Owen of the reversal -- product-features.md's Communications field names only Nadia as the recipient for a reversal; Owen already knows about any dispute he filed with his own card issuer or bank, and this product's evidentiary role is to alert the freelancer, not the client, per XBR-21
- Carrying any action the recipient can take inside the product -- excluded per scope-boundaries.md (SC-18): responding to the dispute happens entirely inside Nadia's own processor account; this email's CTA opens the invoice for her own record-keeping only, never a response flow this product does not offer
- Any in-app channel -- product-features.md's Communications field names only an email to Nadia; this product's in-app notification feed (FEAT-29) is a Later-phase feature not yet built

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always | Nadia works from a laptop or desktop throughout her working day but is not necessarily inside the product at the moment a processor reports a reversal; a dispute has its own response clock in her processor account, so she needs to be alerted the instant it happens, not only the next time she happens to open Clientroom |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Payment reversal recorded | FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | Fires immediately when a reversal commit succeeds | Invoice reference, invoice number, prior status (Paid / Refunded / Partially refunded), amount, currency, client name, project name |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient. She is the account owner and the only party who can act on the dispute in her own processor account; per the Access Matrix, Invoicing & Payments is Full for Nadia and this reversal concerns her own financial record.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional record email per XBR-30 ("transactional emails core to the record... always send") | -- | Always on, no opt-out | N/A -- no preference surface exists for this notification; it is not shown as a toggleable row on FEAT-21.SPEC-002 (Notification Preferences), consistent with the transactional treatment product-features.md applies to record-status emails |

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); this notification is transactional and time-critical -- a chargeback carries its own response deadline in Nadia's processor account, so holding it for a quiet-hours window would shrink her time to respond, directly contradicting the Feature Breakdown Brief's "notified immediately."

## Content Definition

**Email:**
- **Subject:** Payment reversal reported on invoice {invoice_number}
- **Body:**
  Hi {freelancer_first_name},

  Your payment processor has reported a reversal or chargeback on invoice {invoice_number} for {project_name} ({client_name}), amount {amount} {currency}.

  This invoice now shows as Disputed in Clientroom, alongside its original payment record. Respond to the dispute directly in your payment processor account -- Clientroom does not handle chargebacks or disputes itself.
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for the affected invoice

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_first_name} | Freelancer Account -- name | Nadia | Greeting renders as "Hi," |
| {invoice_number} | Invoice -- invoice_number | INV-0042 | Never empty -- invoice_number is required at generation (FEAT-09.SPEC-007) |
| {project_name} | Project -- project_name | Brand Refresh | Never empty -- project_name is required at project creation (FEAT-01.SPEC-002) |
| {client_name} | Client -- client_name | Acme Co. | Never empty -- client_name is required at client creation (FEAT-01.SPEC-001) |
| {amount} | Payment -- amount | 450.00 | Never empty -- this email only fires once a reversal has been correlated to a Succeeded Payment (FEAT-25.SPEC-005) |
| {currency} | Invoice -- currency | USD | Never empty -- currency is required before a project's first invoice (XBR-17) |

## Delivery Rules

**Batching:** None -- each reversal is delivered as its own, individual email the moment it is recorded, regardless of how many other reversals or refunds may be in flight elsewhere in the account; a chargeback is time-sensitive on its own and must never wait to be bundled with another event.
**Deduplication:** At most one email per reversal commit -- FEAT-25.SPEC-005's idempotent handling of a duplicate or out-of-order reversal event for the same invoice means a second delivery of the same underlying processor report produces no second commit, and so no second email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the invoice's Disputed status remains fully visible to her the next time she opens FEAT-09.SPEC-002, regardless of this email's delivery outcome.
**Expiry:** This notification never expires undelivered in the sense of becoming irrelevant -- a reversal is a permanent record and Nadia's dispute-response clock runs in her processor account, not inside this email; retries continue for the full retry window above, and after that the delivery-failure warning (not a silently dropped message) is the surviving signal, alongside the Disputed status itself, which is never hidden.

## Edge Cases

- **Nadia's sign-in email address does not exist or bounces on the first delivery attempt** -- Retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`; after the final failure, she sees a delivery warning inside the product the next time she is in-product, and the invoice's Disputed status is visible regardless.
- **Two reversal notices arrive for the same invoice at effectively the same time (duplicate delivery)** -- FEAT-25.SPEC-005 applies only the first as a genuine commit; the second is an idempotent no-op, so only one email is ever sent for the same underlying event.
- **A reversal is recorded on an invoice Nadia had already partially refunded herself** -- The email still fires normally, naming the invoice's prior status implicitly through its Disputed outcome; the amount named is the Payment's original amount, since that is what the processor is reversing, not the smaller amount Nadia had already refunded.
- **Nadia is offline or away from email when the reversal is recorded** -- The email is queued and delivered as soon as delivery succeeds; there is no in-product-only fallback for this notification, since email is its only channel and the Disputed status is separately always visible whenever she next opens the invoice.
- **The Freelancer Account is deleted (FEAT-24) between the reversal commit and this email's delivery** -- Per FEAT-24's account-removal scope, the delivery is cancelled silently rather than sent to a closed account; there is no freelancer left to notify.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | Triggered by (inbound) | A successful reversal commit fires this notification immediately |
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (outbound) | The CTA deep-links here, showing the Disputed status |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (inbound) | Delivers this notification and reports delivery, bounce, and failure status |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | Triggers (outbound) | A final delivery failure surfaces as a warning to Nadia on the affected project |

## Analytics and Success Signals

- **payment_reversal_notification_delivered** (invoice reference) -- N/A -- no success-metrics.md metric is connected to this feature; retained so delivery of this evidentiary email is observable rather than invisible, per XBR-21's evidentiary intent.
- **payment_reversal_notification_delivery_failed** (final_failure: yes / no) -- N/A -- no success-metrics.md metric is connected to this feature; retained so silent delivery loss of a time-critical dispute alert is observable rather than invisible.

## Acceptance Criteria

**FEAT-25.SPEC-008-AC-01:** Given a reversal is recorded on one of Nadia's invoices, when the commit succeeds, then she receives an email with subject "Payment reversal reported on invoice {invoice_number}" immediately.

**FEAT-25.SPEC-008-AC-02:** Given Nadia opens the reversal email, when she taps "View invoice", then she lands on FEAT-09.SPEC-002 showing the invoice's Disputed status alongside its preserved prior record.

**FEAT-25.SPEC-008-AC-03:** Given the reversal email body, when Nadia reads it, then it states plainly that she must respond to the dispute directly in her payment processor account, since Clientroom does not handle chargebacks or disputes itself.

**FEAT-25.SPEC-008-AC-04:** Given Owen is the client on the reversed invoice, when the reversal is recorded, then he receives no email from this spec.

**FEAT-25.SPEC-008-AC-05:** Given Nadia's email bounces on the first delivery attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears to her in-product.

**FEAT-25.SPEC-008-AC-06:** Given delivery to Nadia fails permanently after retries are exhausted, then she sees a delivery warning on the affected project, and the invoice's Disputed status is still visible whenever she next opens it.

**FEAT-25.SPEC-008-AC-07:** Given two reversal notices arrive for the same invoice at effectively the same time, then only one email is sent, since the second commit is an idempotent no-op.

**FEAT-25.SPEC-008-AC-08:** Given the invoice was already Partially refunded by Nadia's own action before this reversal, when the reversal email is sent, then it still names the Payment's original amount as the reversed amount.

**FEAT-25.SPEC-008-AC-09:** Given a reversal is recorded at any hour, when the email is ready to send, then it sends immediately with no quiet-hours hold, since this notification is transactional and time-critical.

**FEAT-25.SPEC-008-AC-10:** Given Nadia looks for a way to turn this notification off, when she checks FEAT-21.SPEC-002 (Notification Preferences), then no toggle for it exists there.

**FEAT-25.SPEC-008-AC-11:** Given Nadia's Freelancer Account is deleted between the reversal commit and this email's delivery, when the delivery would otherwise fire, then it is cancelled silently.

**FEAT-25.SPEC-008-AC-12:** Given Nadia is away from email when a reversal is recorded, when she next checks her inbox, then the queued email is present with its original content, unmodified by the passage of time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no opt-out) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
