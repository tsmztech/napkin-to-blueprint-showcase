---
document_type: feature-dependency-map
produced_by: requirements-architect
status: final
created: 2026-09-26
feature_count: 33
shared_entity_count: 21
cross_feature_rule_count: 35
---

# Feature Dependency Map

## Features

All 33 features from product-features.md, with Phase, Priority, and Type carried from each feature's Stage 2 entry. "Depends On" is carried from the Feature Interaction Summary in product-features.md; "Depended On By" is its inverse.

| Number | Slug | Name | Priority | Phase | Type | Depends On | Depended On By |
|--------|------|------|----------|-------|------|------------|----------------|
| FEAT-01 | client-project-management | Client & Project Management | Core | MVP | User-Facing | -- | FEAT-02, FEAT-09, FEAT-12, FEAT-18, FEAT-20, FEAT-23, FEAT-28 |
| FEAT-02 | proposal-creation-sending | Proposal Creation & Sending | Core | MVP | User-Facing | FEAT-01, FEAT-15 | FEAT-03, FEAT-14, FEAT-20, FEAT-28 |
| FEAT-03 | proposal-acceptance | Proposal Acceptance | Core | MVP | User-Facing | FEAT-02 | FEAT-04, FEAT-09, FEAT-13, FEAT-14, FEAT-26 |
| FEAT-04 | milestone-payment-schedule-setup | Milestone & Payment Schedule Setup | Core | MVP | User-Facing | FEAT-03 | FEAT-06 |
| FEAT-05 | client-portal-access-magic-link-login | Client Portal Access (Magic-Link Login) | Core | MVP | Platform | FEAT-18 | FEAT-13, FEAT-14, FEAT-27, FEAT-30, FEAT-33 |
| FEAT-06 | deliverable-upload-sharing | Deliverable Upload & Sharing | Core | MVP | User-Facing | FEAT-04, FEAT-16 | FEAT-07, FEAT-08, FEAT-13, FEAT-14, FEAT-17, FEAT-28 |
| FEAT-07 | deliverable-review-feedback | Deliverable Review & Feedback | Core | MVP | User-Facing | FEAT-06 | FEAT-08, FEAT-14 |
| FEAT-08 | milestone-approval | Milestone Approval | Core | MVP | User-Facing | FEAT-06, FEAT-07 | FEAT-09, FEAT-13, FEAT-14, FEAT-30 |
| FEAT-09 | invoice-generation-sending | Invoice Generation & Sending | Core | MVP | User-Facing | FEAT-01, FEAT-03, FEAT-08, FEAT-15, FEAT-21, FEAT-32 | FEAT-10, FEAT-11, FEAT-12, FEAT-13, FEAT-14, FEAT-22, FEAT-25, FEAT-28 |
| FEAT-10 | invoice-payment-processing | Invoice Payment Processing | Core | MVP | User-Facing | FEAT-09, FEAT-32 | FEAT-11, FEAT-12, FEAT-13, FEAT-14, FEAT-22, FEAT-25 |
| FEAT-11 | automated-payment-reminders | Automated Payment Reminders | Core | MVP | Lifecycle | FEAT-09, FEAT-10, FEAT-15 | FEAT-13, FEAT-14 |
| FEAT-12 | freelancer-financial-dashboard | Freelancer Financial Dashboard | Core | MVP | User-Facing | FEAT-01, FEAT-09, FEAT-10, FEAT-15 | -- |
| FEAT-13 | immutable-activity-audit-trail | Immutable Activity & Audit Trail | Core | MVP | Platform | FEAT-03, FEAT-05, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-25, FEAT-31 | FEAT-29, FEAT-31 |
| FEAT-14 | notifications-email | Notifications (Email) | Core | MVP | Platform | FEAT-02, FEAT-03, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-19, FEAT-25, FEAT-31, FEAT-32 | FEAT-21, FEAT-29, FEAT-31, FEAT-33 |
| FEAT-15 | currency-tax-handling | Currency & Tax Handling | Core | MVP | User-Facing | -- | FEAT-02, FEAT-09, FEAT-11, FEAT-12 |
| FEAT-16 | large-file-handling-storage | Large File Handling & Storage | Core | MVP | Platform | -- | FEAT-06, FEAT-17 |
| FEAT-17 | deliverable-version-history | Deliverable Version History | Important | MVP | User-Facing | FEAT-06, FEAT-16 | -- |
| FEAT-18 | client-contact-management-roles | Client Contact Management & Roles | Important | MVP | User-Facing | FEAT-01 | FEAT-05, FEAT-13, FEAT-14 |
| FEAT-19 | freelancer-branding | Freelancer Branding | Important | MVP | User-Facing | -- | FEAT-14, FEAT-20, FEAT-27 |
| FEAT-20 | onboarding-first-run-setup | Onboarding / First-Run Setup | Important | MVP | Lifecycle | FEAT-01, FEAT-02, FEAT-19, FEAT-32, FEAT-33 | FEAT-30 |
| FEAT-21 | settings-account-management | Settings & Account Management | Important | MVP | Lifecycle | FEAT-14 | FEAT-09 |
| FEAT-22 | accounting-export | Accounting Export | Important | MVP | User-Facing | FEAT-09, FEAT-10 | -- |
| FEAT-23 | subscription-plan-billing-management | Subscription Plan & Billing Management | Important | MVP | Lifecycle | FEAT-01 | -- |
| FEAT-24 | data-export-account-deletion | Data Export & Account Deletion | Important | MVP | Lifecycle | All features (FEAT-01 – FEAT-33; see note) | -- |
| FEAT-25 | refund-cancelled-project-handling | Refund & Cancelled Project Handling | Important | MVP | User-Facing | FEAT-09, FEAT-10, FEAT-32 | FEAT-13, FEAT-14 |
| FEAT-26 | legally-binding-e-signature-for-proposals | Legally Binding E-Signature for Proposals | Nice-to-Have | v1 | User-Facing | FEAT-03 | -- |
| FEAT-27 | custom-domain-per-freelancer | Custom Domain per Freelancer | Nice-to-Have | Later | Platform | FEAT-05, FEAT-19 | -- |
| FEAT-28 | global-search-across-clients-projects | Global Search Across Clients & Projects | Nice-to-Have | v1 | User-Facing | FEAT-01, FEAT-02, FEAT-06, FEAT-09 | -- |
| FEAT-29 | in-app-notification-center | In-App Notification Center | Nice-to-Have | Later | Platform | FEAT-13, FEAT-14 | -- |
| FEAT-30 | contextual-help-guidance | Contextual Help & Guidance | Nice-to-Have | Later | Lifecycle | FEAT-05, FEAT-08, FEAT-20 | -- |
| FEAT-31 | operator-support-access | Operator Support Access | Important | MVP | Platform | FEAT-13, FEAT-14 | FEAT-13, FEAT-14 |
| FEAT-32 | payment-account-connection | Payment Account Connection | Core | MVP | Platform | -- | FEAT-09, FEAT-10, FEAT-14, FEAT-20, FEAT-25 |
| FEAT-33 | portal-referral-attribution | Portal Referral Attribution | Important | MVP | Lifecycle | FEAT-05, FEAT-14 | FEAT-20 |

Note: Data Export & Account Deletion (FEAT-24) reads from, and on deletion removes records of, every data-holding feature (product-features.md: "FEAT-01 through FEAT-33"). It is not repeated in every "Depended On By" cell; treat every feature that stores freelancer or client data as depended on by FEAT-24.

## Shared Data Entities

Every entity in the Domain Entity Inventory referenced by two or more features (21 of 23). Accounting Export File (FEAT-22 only) and Data Export Archive (FEAT-24 only) are single-feature entities and are not listed. Roles named in Contention lines are the four Access Matrix roles in user-persona.md: Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), and Dana (Support Operator). Dana is read-only everywhere (ASMP-18) and never contends for a write. Data Export & Account Deletion (FEAT-24) reads and, on account deletion, removes every entity below; that is not repeated per entity.

### Client

- **Lifecycle:** Created by FEAT-01. Read by FEAT-02, FEAT-12, FEAT-18, FEAT-23, FEAT-28, FEAT-31. Updated by FEAT-01 (rename, billing details). Archived by FEAT-01. Deleted by FEAT-01 (only while it has no sent proposal, invoice, or activity) and by FEAT-24 (account deletion).
- **Fields (functional):**
  - client_name -- company name (required)
  - billing_name -- name printed on invoices (required before the first invoice for this client is sent)
  - billing_address -- address printed on invoices (required before the first invoice is sent)
  - tax_id -- client's tax identifier (optional)
  - status -- Active or Archived (required, defaults to Active)
  - currency and tax treatment -- per client/project billing currency and tax label/rate, set through FEAT-15 (required before the first invoice)
- **Relationships:** Belongs to one Freelancer Account. Has many Projects and many Client Contacts. Counts toward the Subscription Plan's active-client limit while Active.
- **Contention:** Low — only Nadia (FEAT-01) edits client records; FEAT-15 edits the currency/tax fields and FEAT-23 only reads the active count. The same freelancer editing in two open sessions resolves last-write-wins on descriptive fields (name, billing details); archive and delete re-check open items (unpaid invoices, pending approvals, any sent record) at the moment of commit and reject-with-refresh if the state changed.
- **Data Sensitivity:** Business contact and billing data about a client company; billing name and address may identify a sole trader and are treated as GDPR-class personal data (ASMP-24). Strictly isolated per freelancer and never visible to other clients (ASMP-23).
- **Source:** Domain Entity Inventory, product-features.md

### Client Contact

- **Lifecycle:** Created by FEAT-18 (by Nadia, or by Owen inviting a Reviewer). Read by FEAT-02, FEAT-03, FEAT-05, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-14, FEAT-31. Updated by FEAT-18 (role change) and FEAT-05 (login timestamps). Removed by FEAT-18 (access revoked, details erased on request). Deleted by FEAT-24.
- **Fields (functional):**
  - name -- contact's name (required)
  - email -- sign-in and notification address (required, unique within the client company)
  - role -- Primary or Reviewer (required)
  - invited_by -- Nadia or the inviting Primary contact (required)
  - status -- Invited, Active, or Removed
  - last_sign_in -- login timestamps captured by FEAT-05
  - help-tip dismissals -- dismissal state held with the contact (FEAT-30, Later)
- **Relationships:** Belongs to one Client. Authenticates into that client's portal only (FEAT-05). Is the actor on acceptances (FEAT-03), approvals (FEAT-08), comments (FEAT-07), and payments (FEAT-10). A person who is a contact for several freelancers holds a separate Client Contact per freelancer.
- **Contention:** Nadia (FEAT-18, Full) and Owen (FEAT-18, Own-only: invite Reviewer contacts) can both add contacts to the same client concurrently. Adding a contact whose email already exists in that client is rejected-with-refresh (email is unique per client). Role changes and removals are Nadia-only; removing the last Primary is re-checked at commit and rejected if no other Primary exists at that moment. A role change applies to future actions only.
- **Data Sensitivity:** Personal data — name and email of individuals at client companies worldwide; GDPR-class (ASMP-24). An erasure request removes contact details but keeps acceptances and approvals under the contact's name as evidence (ASMP-20). Never visible to another client company (ASMP-23).
- **Source:** Domain Entity Inventory, product-features.md

### Project

- **Lifecycle:** Created by FEAT-01. Read by FEAT-02, FEAT-04, FEAT-05, FEAT-12, FEAT-13, FEAT-15, FEAT-28, FEAT-31. Updated by FEAT-01 (rename, mark complete), FEAT-15 (currency and tax treatment), FEAT-25 (mark cancelled). Archived by FEAT-01. Deleted by FEAT-24 only (no in-product project delete).
- **Fields (functional):**
  - project_name -- (required)
  - client -- the owning Client (required, exactly one)
  - stage -- derived label (Draft, In Progress, Complete, Cancelled, Archived) computed from proposal, milestone, and invoice state
  - currency -- billing currency (required before first invoice; fixed once the first invoice is sent)
  - tax_label and tax_rate -- freelancer-configured tax line, or none
  - completed_at / cancelled_at -- timestamps of those transitions
- **Relationships:** Belongs to one Client. Has at most one active (non-voided) Proposal, one Payment Schedule, many Milestones, Deliverables, Invoices, and Activity Log Entries.
- **Contention:** Nadia is the only writer, across FEAT-01, FEAT-15, and FEAT-25; client contacts only read. Two sessions of the same freelancer could race on complete/cancel/archive — these transitions are rejected-with-refresh if the project's state changed since it was loaded; renames are last-write-wins. System-driven stage changes (acceptance, approval) never overwrite a freelancer's explicit Complete or Cancelled.
- **Data Sensitivity:** Business-confidential work information of the freelancer and client; no special-category data. Isolated per client (ASMP-23).
- **Source:** Domain Entity Inventory, product-features.md

### Proposal

- **Lifecycle:** Created by FEAT-02. Read by FEAT-03, FEAT-04, FEAT-13, FEAT-28, FEAT-31. Updated by FEAT-02 (edit before acceptance, which voids and re-sends), FEAT-03 (accepted state), FEAT-26 (signature record, v1). Deleted by FEAT-02 (unsent drafts only) and FEAT-24.
- **Fields (functional):**
  - scope_description -- (required)
  - price -- positive amount in the project currency (required)
  - currency -- from the project (FEAT-15)
  - payment_schedule_reference -- link to the project's Payment Schedule (FEAT-04)
  - status -- Draft, Sent, Voided, or Accepted
  - sent_at -- send timestamp; copied_from -- earlier proposal used as a starting point (optional)
  - accepted_at and accepted_by -- written once on acceptance, never altered
  - signature record -- signer identity, signature data, timestamp (FEAT-26, v1)
- **Relationships:** Belongs to one Project. Sent to the client's Primary contact(s). Referenced by the Payment Schedule. Carries request-changes Comments from Owen (FEAT-03).
- **Contention:** Nadia (FEAT-02: edit, void and re-send) can act while Owen (FEAT-03: accept or request changes) is viewing the same proposal. Resolution is reject-with-refresh: an Accept against a version that was voided in the meantime is refused and Owen is shown the current version; an edit Nadia starts after acceptance is refused because an accepted proposal is immutable. Acceptance is recorded exactly once, so two Primary contacts accepting at the same moment yield one acceptance and the second sees "already accepted."
- **Data Sensitivity:** Commercially confidential (scope and price) and evidentiary; accepted content is immutable (ASMP-15, ASMP-25). Accepting contact's identity is personal data (ASMP-24). Reviewer contacts cannot see proposal content (Access Matrix).
- **Source:** Domain Entity Inventory, product-features.md

### Milestone

- **Lifecycle:** Created by FEAT-04. Read by FEAT-05, FEAT-06, FEAT-07, FEAT-09, FEAT-13, FEAT-31. Updated by FEAT-04 (rename, re-price, re-order, target date — not once approved or invoiced), FEAT-06 (Deliverable Uploaded state), FEAT-08 (Approved, Reopened). Deleted by FEAT-04 (only if never approved or invoiced) and FEAT-24.
- **Fields (functional):**
  - name -- (required)
  - order -- position within the project (required)
  - price or no_separate_charge flag -- one of the two is required
  - payment_trigger -- whether approval issues an invoice (per the Payment Schedule)
  - target_date -- optional; shown in each viewer's time zone (FEAT-15)
  - status -- Defined, Deliverable Uploaded, Approved, Reopened
  - approved_at and approved_by -- written once on approval, never altered; reopen events are separate logged records
- **Relationships:** Belongs to one Project; governed by its Payment Schedule. Has many Deliverables and milestone-level Comments. Approval creates the next Invoice (FEAT-08 → FEAT-09).
- **Contention:** Nadia (FEAT-04: adjust schedule; FEAT-08: reopen) and Owen (FEAT-08: approve) can act on the same milestone concurrently. Resolution is reject-with-refresh: an approval is recorded only against the state Owen was shown — if Nadia re-priced or removed the milestone in the meantime, the approval is refused and the refreshed milestone is shown; once approved, Nadia's edit is refused and she must reopen. Approval is exactly-once.
- **Data Sensitivity:** Business-confidential pricing; approval records are evidentiary and immutable (ASMP-15, ASMP-25); the approver's identity is personal data (ASMP-24).
- **Source:** Domain Entity Inventory, product-features.md

### Payment Schedule

- **Lifecycle:** Created by FEAT-04. Read by FEAT-02 (proposal references it), FEAT-03 (deposit trigger), FEAT-08 (next-invoice trigger), FEAT-01 (on-completion trigger), FEAT-09. Updated by FEAT-04 (dated, non-retroactive adjustments). Deleted by FEAT-24.
- **Fields (functional):**
  - structure -- deposit, per-milestone, on completion, or a mix (required; at least one payment trigger must exist to invoice at all)
  - deposit_amount -- when the structure includes a deposit
  - completion_amount -- when the structure includes an on-completion payment
  - change_history -- dated record of mid-project adjustments
- **Relationships:** One per Project. Attached to the accepted Proposal. Drives Invoice Generation (FEAT-09) through its triggers.
- **Contention:** Only Nadia edits it (FEAT-04), but her edits can race with triggers firing from Owen's actions (acceptance in FEAT-03, approval in FEAT-08). Resolution: a trigger uses the schedule as it stood at the moment of the triggering action; an adjustment saved afterward is dated and applies to later triggers only (never retroactive). Concurrent edits by the same freelancer in two sessions resolve reject-with-refresh.
- **Data Sensitivity:** Business-confidential pricing terms; no personal data.
- **Source:** Domain Entity Inventory, product-features.md

### Deliverable

- **Lifecycle:** Created by FEAT-06. Read by FEAT-05, FEAT-07, FEAT-08, FEAT-17, FEAT-28, FEAT-31. Updated by FEAT-06 (replace, link check) and FEAT-16 (file storage state). Removed by FEAT-06 (withdraw; not allowed on an approved milestone, always logged). Deleted by FEAT-24.
- **Fields (functional):**
  - kind -- uploaded file or linked external asset (design-tool, cloud-drive, or file-sharing link)
  - file or link -- the uploaded file, or a valid, reachable link (not copied in)
  - milestone -- owning Milestone (required)
  - uploaded_at -- (required); size -- for uploaded files, within the per-file ceiling
  - status -- Uploading, Active, Superseded, Removed; link_status -- reachable or flagged
  - first_client_view_at -- timestamp of a client contact's first view (recorded in FEAT-13)
- **Relationships:** Belongs to one Milestone. Has one or more Deliverable Versions. Has many pinned Comments.
- **Contention:** Nadia is the only writer (FEAT-06/FEAT-16/FEAT-17); Owen and Priya view. The contended moment is removal or replacement while a client is viewing or while Owen approves the milestone: removal of a deliverable is re-checked at commit and rejected if the milestone has been approved in the meantime (it can then only be superseded); a client viewing a replaced deliverable is refreshed to the latest version.
- **Data Sensitivity:** Client-confidential work product (design files, videos, documents) that may itself contain personal data; strictly isolated per client (ASMP-23); the operator can list files but never download them (ASMP-18).
- **Source:** Domain Entity Inventory, product-features.md

### Deliverable Version

- **Lifecycle:** Created by FEAT-06 (first upload), FEAT-17 (re-uploads), FEAT-16 (stored file). Read by FEAT-07, FEAT-17, FEAT-31. Never updated (immutable once uploaded). Never deleted in-product; removed only by FEAT-24.
- **Fields (functional):**
  - round_number -- sequential per deliverable (required)
  - file -- the uploaded file for this round (required)
  - uploaded_at -- (required)
  - is_latest -- derived: the most recent upload
- **Relationships:** Belongs to one Deliverable. Anchors version-specific Comments. Counts against the freelancer's storage allowance (FEAT-16).
- **Contention:** None — versions are immutable once uploaded and only Nadia creates them; a new round is appended rather than editing an existing one, so no concurrent modification of a version is possible.
- **Data Sensitivity:** Same as Deliverable: client-confidential work product, strictly isolated per client (ASMP-23); retained for the life of the account (ASMP-22).
- **Source:** Domain Entity Inventory, product-features.md

### Comment

- **Lifecycle:** Created by FEAT-07 (deliverable, version, and milestone comments; freelancer replies) and FEAT-03 (request-changes notes on a proposal). Read by FEAT-02, FEAT-08, FEAT-17, FEAT-31. Updated by FEAT-07 (edit within a short grace window; retraction as a soft removal). Deleted by FEAT-24 only.
- **Fields (functional):**
  - text -- 1–2,000 characters (required)
  - author -- client contact or Nadia (required)
  - posted_at -- (required)
  - target -- a deliverable version, a milestone as a whole, or a proposal (change request) (required)
  - reply_to -- parent comment in the thread (optional)
  - status -- Posted or Retracted
- **Relationships:** Pinned to a Deliverable Version, a Milestone, or a Proposal. Visible to all contacts of the same client (except proposal notes, which Reviewers cannot see) and to Nadia.
- **Contention:** None — each comment is written and retracted only by its own author, so two actors never modify the same comment; many authors (Owen, Priya, Nadia) add comments to the same thread concurrently, which is append-only and needs no resolution beyond ordering by posted time.
- **Data Sensitivity:** Personal data (author identity, free text that may mention individuals), GDPR-class (ASMP-24); included in the freelancer's export and removed on account deletion; strictly isolated per client (ASMP-23).
- **Source:** Domain Entity Inventory, product-features.md

### Invoice

- **Lifecycle:** Created by FEAT-09 (automatic from FEAT-03 deposit, FEAT-08 approval, FEAT-01 completion; ad hoc; credit notes). Read by FEAT-10, FEAT-11, FEAT-12, FEAT-13, FEAT-15, FEAT-22, FEAT-28, FEAT-31. Updated by FEAT-09 (due date before sending; send), FEAT-10 (Payment pending, Paid), FEAT-11 (Overdue flag, reminder pause), FEAT-15 (currency and tax fields at generation), FEAT-25 (Refunded, Partially refunded, Disputed). Never edited after sending; corrected only by a credit note or new invoice. Deleted by FEAT-24 (subject to legal retention).
- **Fields (functional):**
  - invoice_number -- unique and sequential per freelancer (required)
  - project and triggering_event -- deposit, milestone approval, completion, ad hoc, or correction (required)
  - amount, currency, tax_label, tax_rate, total -- (required; amount matches the trigger)
  - freelancer business details and client billing details -- (required, from FEAT-21 and FEAT-01)
  - issue_date and due_date -- (required; due date from default payment terms, adjustable before sending)
  - status -- Generated, Sent, Payment pending, Paid, Paid (recorded by freelancer), Overdue, Refunded, Partially refunded, Disputed, Corrected
  - pay_link availability -- derived from Payment Account Connection (FEAT-32)
  - reminder_paused -- (FEAT-11)
- **Relationships:** Belongs to one Project; addressed to the client's Primary contact(s). Has at most one successful Payment (no partial payments). Has a Reminder Log. May be corrected by a credit note Invoice.
- **Contention:** Several writers act on one invoice: Owen paying (FEAT-10), Nadia recording an off-platform payment (FEAT-10) or marking a refund (FEAT-25), the reminder schedule (FEAT-11), and processor status reports (FEAT-10/FEAT-25 via FEAT-32). Resolution: status changes are reject-with-refresh against the current status — an invoice already Paid refuses a second payment or a manual "mark paid"; a refund cannot exceed the amount paid; reminders re-check paid/paused/pending status immediately before sending and skip if it changed. Processor-confirmed payment status is authoritative over a concurrent manual entry.
- **Data Sensitivity:** Financial records with personal data (client billing name and address, contact identity), GDPR-class (ASMP-24); evidentiary and immutable once sent (ASMP-15, ASMP-25); may be subject to legal financial-record retention on account deletion (SC-24). No card data is ever held (ASMP-24). Hidden from Reviewer contacts.
- **Source:** Domain Entity Inventory, product-features.md

### Payment

- **Lifecycle:** Created by FEAT-10 (processor-confirmed card or bank-transfer payment, or a manually recorded off-platform payment). Read by FEAT-12, FEAT-22, FEAT-25, FEAT-31. Updated by FEAT-10 (Pending → Succeeded or Failed) and FEAT-25 (Reversed, refunded amount). Never deleted in-product; removed by FEAT-24 subject to legal retention.
- **Fields (functional):**
  - invoice -- (required)
  - amount -- full invoice amount (required; no partial payments)
  - method -- card, bank transfer, or recorded off-platform method (required)
  - paid_at -- (required; a manual record cannot be future-dated)
  - status -- Initiated, Pending, Succeeded, Failed, Reversed, Recorded manually
  - recorded_by -- Nadia, for manual records
- **Relationships:** Belongs to one Invoice. Lands directly in the freelancer's own processor account via her Payment Account Connection; the product never holds the funds.
- **Contention:** Owen (paying via FEAT-10) and Nadia (recording an off-platform payment in FEAT-10) could act on the same invoice at once, and processor status updates (pending → succeeded/failed, reversal) arrive asynchronously. Resolution: reject-with-refresh on the invoice's paid status — the first confirmed full payment wins and later attempts are refused with the updated status; processor-reported status updates are applied in order and are authoritative.
- **Data Sensitivity:** Financial record; no card numbers or bank credentials are ever captured (ASMP-24, SC-10); payer identity is personal data (ASMP-24); may be subject to legal financial-record retention (SC-24).
- **Source:** Domain Entity Inventory, product-features.md

### Reminder Log

- **Lifecycle:** Created by FEAT-11. Read by FEAT-09 (reminder history on invoice detail), FEAT-13, FEAT-31. Updated by FEAT-11 (Sent, Paused, Resumed). Deleted by FEAT-24.
- **Fields (functional):**
  - invoice -- (required)
  - reminder_type -- day 3, day 10, or manual
  - scheduled_for and sent_at -- in the freelancer's time zone
  - pause_state -- Active, Paused by freelancer, Paused while bank transfer pending
- **Relationships:** Belongs to one Invoice; each send writes an Activity Log Entry and a Notification.
- **Contention:** Nadia (pause/resume, manual reminder) and the automatic schedule (FEAT-11) act on the same log. Resolution: the schedule re-checks pause, pending, and paid state immediately before each send; a pause saved first wins; manual reminders are limited to one per invoice per day, so a second concurrent manual send is refused with refresh.
- **Data Sensitivity:** Low — send timestamps and pause state; linked to the recipient contact, which is personal data (ASMP-24).
- **Source:** Domain Entity Inventory, product-features.md

### Activity Log Entry

- **Lifecycle:** Created by FEAT-13 on behalf of FEAT-03, FEAT-05, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-25, FEAT-31. Read by FEAT-13, FEAT-29 (Later), FEAT-24 (included in the archive), FEAT-31. Never updated. Never deleted in-product; removed only by FEAT-24 account deletion.
- **Fields (functional):**
  - event_type -- e.g., proposal accepted, milestone approved, invoice sent, deliverable uploaded or removed, first client view, reminder sent, contact role changed, refund/reversal/cancellation, manual payment, support session (required)
  - actor -- the freelancer, a named client contact, the operator, or the product itself (required)
  - occurred_at -- (required)
  - affected_record -- reference to the record concerned (required)
  - project -- (required where the event belongs to a project)
- **Relationships:** Belongs to one Freelancer Account and, usually, one Project. References the record it describes.
- **Contention:** None — entries are append-only and never edited or deleted by anyone (ASMP-15); concurrent writers only append independent entries.
- **Data Sensitivity:** Contains actor identities (personal data, GDPR-class, ASMP-24) and evidentiary content; immutable (ASMP-25); visible to Nadia and read-only to Dana; retained for the life of the account (SC-24); a contact's erasure keeps their name on evidence entries (ASMP-20).
- **Source:** Domain Entity Inventory, product-features.md

### Branding Profile

- **Lifecycle:** Created by FEAT-19 (or offered during FEAT-20). Read by FEAT-02, FEAT-05, FEAT-06, FEAT-09, FEAT-14, FEAT-27, FEAT-31, FEAT-33. Updated by FEAT-19 (set, change, reset to default). Deleted by FEAT-24.
- **Fields (functional):**
  - logo -- image within size and format limits (optional; neutral default when unset)
  - brand_colour -- one primary colour (optional; adjusted for legibility when needed)
- **Relationships:** One per Freelancer Account. Applied to every client-facing screen and email.
- **Contention:** None — only Nadia edits her own branding (FEAT-19, or its step in FEAT-20); a save from a second session simply replaces the first (last-write-wins), and client contacts only view it.
- **Data Sensitivity:** None — a logo and a colour are public-facing brand assets with no personal data.
- **Source:** Domain Entity Inventory, product-features.md

### Subscription Plan

- **Lifecycle:** Created by FEAT-23 (free tier, automatically at sign-up). Read by FEAT-01 (client-count limit check), FEAT-31. Updated by FEAT-23 (upgrade, downgrade, cancel, lapse). Deleted by FEAT-24.
- **Fields (functional):**
  - tier -- Free or Paid (required)
  - billing_cycle -- monthly or yearly (paid only)
  - active_client_count -- derived from FEAT-01
  - status -- Active, Charge failed, Cancelled (ends at period end), Lapsed
- **Relationships:** One per Freelancer Account. Limits how many Active Clients FEAT-01 allows.
- **Contention:** Nadia changing her plan (FEAT-23) can race with her adding a client (FEAT-01) and with subscription-billing status reports. Resolution: the client-limit check runs at the moment a client is added or reactivated against the current plan state (reject-with-refresh, showing the upgrade prompt); billing-capability status reports are authoritative for charge outcomes.
- **Data Sensitivity:** Freelancer's own billing relationship with Clientroom; payment details are held by the subscription-billing capability, never by the product (ASMP-24).
- **Source:** Domain Entity Inventory, product-features.md

### Custom Domain Record

- **Lifecycle:** Created by FEAT-27 (Later). Read by FEAT-05, FEAT-31. Updated by FEAT-27 (verification state). Deleted by FEAT-27 (removing the domain) and FEAT-24.
- **Fields (functional):**
  - domain_name -- (required; one per freelancer)
  - verification_state -- Added, Verifying, Verified, Verification Failed, with a specific failure reason
- **Relationships:** One per Freelancer Account at most. Determines the address at which that freelancer's portal (FEAT-05) is served; the shared default address always remains available.
- **Contention:** None — only Nadia configures it and the verification capability reports its state; there is no second human writer.
- **Data Sensitivity:** None — a domain name is public information.
- **Source:** Domain Entity Inventory, product-features.md

### Freelancer Account

- **Lifecycle:** Created by FEAT-20 (sign-up). Read by FEAT-09 (business details, payment terms), FEAT-14 (notification preferences), FEAT-23, FEAT-31, FEAT-33. Updated by FEAT-21 (profile, business details, payment terms, notification preferences, sign-in email, sessions). Deleted by FEAT-24.
- **Fields (functional):**
  - name and sign-in email -- (required; email changes require re-verification)
  - business_name, business_address, tax_id -- (required before the first invoice is sent)
  - default_payment_terms -- e.g., due on receipt or within N days
  - time_zone -- (FEAT-15)
  - notification_preferences -- optional notifications only; transactional record emails cannot be disabled
  - signed-in devices -- list with sign-out-others
  - help-tip dismissals -- (FEAT-30, Later)
- **Relationships:** Owns every Client, Project, Branding Profile, Subscription Plan, Payment Account Connection, Referral Attribution, and Activity Log Entry in the account.
- **Contention:** None across roles — only Nadia modifies her own account; Dana views read-only. Two open sessions of Nadia's resolve last-write-wins per field, except the sign-in email change, which requires re-verification before it takes effect.
- **Data Sensitivity:** Personal data of the freelancer (name, email, business address, tax ID), GDPR-class (ASMP-24); exportable and deletable on request (ASMP-23); sign-in credentials never visible to the operator.
- **Source:** Domain Entity Inventory, product-features.md

### Notification

- **Lifecycle:** Created by FEAT-14 for triggering events from FEAT-02, FEAT-03, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-20, FEAT-21, FEAT-23, FEAT-24, FEAT-25, FEAT-27, FEAT-31, FEAT-32. Read by FEAT-29 (Later), FEAT-21 (preferences), FEAT-31 (delivery warnings). Updated by FEAT-14 (delivery status). Deleted by FEAT-24.
- **Fields (functional):**
  - notification_type -- exactly one triggering event per type (required)
  - recipient -- a contact or the freelancer entitled to the event per the Access Matrix (required)
  - sent_at -- (required)
  - delivery_status -- Queued, Sent, Delivered, Failed, Bounced (required)
- **Relationships:** References its triggering event and the affected project; a failure surfaces as a delivery warning on that project.
- **Contention:** None — created and updated only by the email delivery process from delivery-status reports; no person edits a notification.
- **Data Sensitivity:** Personal data (recipient email and name, message content about a project), GDPR-class (ASMP-24); content respects role scope (a Reviewer never receives invoice content).
- **Source:** Domain Entity Inventory, product-features.md

### Payment Account Connection

- **Lifecycle:** Created by FEAT-32 (also offered in FEAT-20). Read by FEAT-09 (pay link), FEAT-10, FEAT-25 (reversal notices), FEAT-31 (status only). Updated by FEAT-32 (reconnect; status reported by the payment-processing capability). Deleted by FEAT-32 (disconnect) and FEAT-24 (disconnect on account deletion).
- **Fields (functional):**
  - processor_account_reference -- a reference only; never card numbers or bank credentials (required when connected)
  - status -- Not connected, Connected ("Ready to accept payments"), Needs attention (with the processor's specific reason), Disconnected
  - available_payment_methods -- derived: card and/or bank transfer currently available on pay links
- **Relationships:** One per Freelancer Account. Every Invoice's pay link and every in-portal Payment depends on it.
- **Contention:** Nadia (connect, reconnect, disconnect) and asynchronous status reports from the payment-processing capability both change it. Resolution: processor-reported status is authoritative; a disconnect issued while a client is paying does not cancel a payment already submitted to the processor, and pay links opened afterward show "online payment is temporarily unavailable" (reject-with-refresh on the pay page).
- **Data Sensitivity:** Financial-account linkage; the product stores only a reference and readiness status, never card or bank credentials (ASMP-24, SC-10); the operator sees status only.
- **Source:** Domain Entity Inventory, product-features.md

### Support Access Session

- **Lifecycle:** Created by FEAT-31 (support request, then session). Read by FEAT-13 (session entries in the trail), FEAT-31. Updated by FEAT-31 (Opened → Closed, including automatic close on inactivity). Never edited after closing. Deleted by FEAT-24.
- **Fields (functional):**
  - freelancer_account -- the one account being viewed (required)
  - request_text -- Nadia's support request (required)
  - operator -- Dana's identity (required)
  - opened_at and closed_at -- (required once opened/closed)
- **Relationships:** Belongs to one Freelancer Account; writes Activity Log Entries (FEAT-13) and triggers Notifications to Nadia (FEAT-14).
- **Contention:** None — only Dana opens and closes a session, one account at a time, and the record is never edited after it closes; Nadia only reads it.
- **Data Sensitivity:** Contains the freelancer's support request text and the operator's identity (personal data, ASMP-24); the session itself grants read-only visibility of the account's data, so it is always logged and announced (ASMP-18, ASMP-23).
- **Source:** Domain Entity Inventory, product-features.md

### Referral Attribution

- **Lifecycle:** Created by FEAT-33 at sign-up (answer captured in FEAT-20). Read by FEAT-33 (aggregate growth measurement only). Never updated. Deleted by FEAT-24 with the account.
- **Fields (functional):**
  - referring_portal -- reference to the referring freelancer account, or unknown (optional)
  - self_reported_source -- the optional "How did you hear about us?" answer, or unknown
  - recorded_at -- (required)
- **Relationships:** Belongs to the new Freelancer Account; references the referring Freelancer Account without exposing any of its client data.
- **Contention:** None — recorded once at sign-up and never edited.
- **Data Sensitivity:** Low personal data (a link between two freelancer accounts and a free-text answer); used only in aggregate — no persona browses it, and the referring freelancer is never told who signed up (FEAT-33 Access).
- **Source:** Domain Entity Inventory, product-features.md

## Navigation Connections

Derived from the journey steps in user-journeys.md and the Interactions fields in product-features.md.

| From Feature | From Context | To Feature | To Context | Trigger |
|-------------|-------------|------------|-----------|---------|
| FEAT-33 | "Made with Clientroom" mark on a portal page or email | FEAT-20 | sign-up and first-run setup (referring portal recorded) | Visitor follows the referral mark and chooses to sign up |
| FEAT-33 | public product page | FEAT-05 | portal home | Client contact returns to the portal in one step |
| FEAT-20 | guided setup: first client and project | FEAT-01 | add client / create project | Onboarding step "Add first client and project" |
| FEAT-20 | guided setup: branding step | FEAT-19 | branding settings | Onboarding step "Set branding" (skippable) |
| FEAT-20 | guided setup: connect payments | FEAT-32 | connect payment account | Onboarding step "Connect payments" (optional) |
| FEAT-20 | guided setup: first proposal | FEAT-02 | proposal draft for the new project | Onboarding step "Draft the first proposal" |
| FEAT-20 | onboarding complete | FEAT-12 | freelancer dashboard | First client, project, and draft proposal exist |
| FEAT-21 | settings | FEAT-19 | branding settings | Nadia returns to a skipped branding step later |
| FEAT-21 | settings: close account | FEAT-24 | data export and account deletion | Nadia chooses to close her account |
| FEAT-01 | client and project roster | FEAT-01 | project view | Nadia opens a project |
| FEAT-01 | project view (proposal area) | FEAT-02 | proposal draft / proposal detail | Nadia drafts, edits, or opens the project's proposal |
| FEAT-01 | project view (milestones area) | FEAT-04 | milestone and payment schedule editor | Nadia defines or adjusts milestones and payment triggers |
| FEAT-01 | project view (milestone) | FEAT-06 | deliverable upload on the milestone | Nadia uploads or links a deliverable |
| FEAT-01 | project view (invoices area) | FEAT-09 | invoice detail / ad-hoc invoice | Nadia opens an invoice or issues one outside the schedule |
| FEAT-01 | project view (activity) | FEAT-13 | project activity trail | Nadia opens the project's trail |
| FEAT-01 | client detail | FEAT-18 | client contact list | Nadia manages the client's contacts and roles |
| FEAT-01 | project billing setup | FEAT-15 | project currency and tax line | Nadia sets currency and tax before the first invoice |
| FEAT-01 | add client beyond the free-tier limit | FEAT-23 | upgrade prompt and plan subscription | Adding an active client exceeds the plan's limit |
| FEAT-01 | mark project complete | FEAT-09 | on-completion invoice | Completion triggers the final invoice when the schedule includes one |
| FEAT-02 | proposal (sent) | FEAT-18 | client contact list | Sending is blocked until the client has a Primary contact |
| FEAT-02 | proposal-sent email | FEAT-05 | magic-link sign-in → proposal view | Owen opens the proposal link and signs in |
| FEAT-05 | portal home | FEAT-03 | proposal review and accept | Owen opens the proposal waiting on him |
| FEAT-03 | request-changes note | FEAT-02 | proposal detail with the change request | Nadia opens the change-request email and revises the proposal |
| FEAT-03 | acceptance recorded | FEAT-09 | deposit invoice | Acceptance triggers the deposit invoice when the schedule includes one |
| FEAT-09 | invoice email / invoice view | FEAT-10 | pay invoice | Owen follows the pay link |
| FEAT-06 | deliverable-ready email | FEAT-05 | magic-link sign-in → deliverable view | A contact opens the deliverable notification |
| FEAT-05 | portal home | FEAT-07 | deliverable review and comment thread | Priya or Owen opens a deliverable waiting for review |
| FEAT-07 | deliverable view | FEAT-17 | version selector / earlier round | A viewer opens an earlier version of the deliverable |
| FEAT-07 | comment notification email | FEAT-07 | deliverable comment thread (freelancer side) | Nadia opens the comment email to reply |
| FEAT-07 | milestone view (deliverables and comments) | FEAT-08 | approve milestone | Owen reviews the round and chooses Approve |
| FEAT-08 | approval recorded | FEAT-09 | next invoice | Approval issues the next invoice in the schedule |
| FEAT-08 | approved milestone (freelancer side) | FEAT-08 | reopen milestone | Nadia reopens an approved milestone |
| FEAT-18 | invitation email | FEAT-05 | first magic-link sign-in | A newly added or invited contact signs in for the first time |
| FEAT-05 | portal home (Primary contact) | FEAT-18 | invite a Reviewer colleague | Owen invites a colleague from his portal view |
| FEAT-05 | portal home | FEAT-10 | invoice list / pay invoice | Owen opens an invoice waiting on him |
| FEAT-05 | expired or invalid link page | FEAT-05 | request a fresh sign-in link | Contact requests a new link in one step |
| FEAT-11 | reminder email | FEAT-10 | pay invoice | Owen pays from the reminder's link |
| FEAT-12 | dashboard | FEAT-11 | overdue invoice detail with reminder history / pause | Nadia opens an overdue invoice |
| FEAT-12 | dashboard aggregate totals | FEAT-12 | client or project financial drill-down | Nadia drills into one client |
| FEAT-12 | client drill-down | FEAT-09 | invoice detail | Nadia opens a specific invoice |
| FEAT-12 | dashboard | FEAT-22 | accounting export | Nadia generates the month's export |
| FEAT-12 | empty dashboard | FEAT-02 | proposal draft | Zero-state prompt toward sending a first proposal |
| FEAT-13 | activity trail entry | FEAT-13 | printable record copy | Nadia shares a record with a client |
| FEAT-13 | activity trail (dispute) | FEAT-25 | mark invoice refunded / project cancelled | Nadia records the outcome of a dispute |
| FEAT-09 | invoice detail (freelancer side) | FEAT-10 | record an off-platform payment | Nadia marks an invoice paid elsewhere |
| FEAT-09 | invoice detail (freelancer side) | FEAT-25 | mark refunded / partial refund | Nadia records a refund issued through her processor |
| FEAT-09 | invoice issued without a connected account | FEAT-32 | connect payment account | Prompt to connect so pay links work |
| FEAT-32 | "needs attention" notice | FEAT-31 | contact support | Nadia cannot resolve the connection problem herself |
| FEAT-31 | support-session notice email | FEAT-13 | activity trail (support session entries) | Nadia checks who looked at her account |
| FEAT-14 | delivery warning on a project | FEAT-18 | client contact list | Nadia corrects a bouncing contact email |
| FEAT-16 | storage-limit warning | FEAT-23 | plan view | Nadia reviews plan and usage when near her storage allowance |
| FEAT-28 | search results (v1) | FEAT-01 / FEAT-02 / FEAT-06 / FEAT-09 | client, project, proposal, deliverable, or invoice | Nadia selects a search result |
| FEAT-29 | notification feed item (Later) | FEAT-01 | project view | Nadia opens the related project from the feed |
| FEAT-19 | branding settings | FEAT-27 | custom domain setup (Later) | Nadia goes further than logo and colour |

## Cross-Feature Business Rules

| Rule ID | Description | Affected Features | Authority |
|---------|-------------|-------------------|-----------|
| XBR-01 | Accepting a proposal immediately generates and sends a deposit invoice when the project's payment schedule includes a deposit, using the schedule as it stood at acceptance. | FEAT-03, FEAT-04, FEAT-09 | FEAT-09 (owns invoice creation; FEAT-03 fires the trigger) |
| XBR-02 | Approving a milestone automatically generates and sends the next invoice in the payment schedule, with no freelancer action. | FEAT-08, FEAT-04, FEAT-09 | FEAT-09 (owns invoice creation; FEAT-08 fires the trigger) |
| XBR-03 | Marking a project complete generates the on-completion invoice when the schedule includes one; completed projects stay visible to the client until archived. | FEAT-01, FEAT-04, FEAT-09 | FEAT-09 (owns invoice creation; FEAT-01 fires the trigger) |
| XBR-04 | Evidence records are never silently altered: an accepted proposal, a milestone approval, a sent invoice, and every trail entry are immutable; changes happen only as new, logged events (proposal void-and-resend before acceptance, milestone reopen, credit note or new invoice, refund/reversal status). | FEAT-02, FEAT-03, FEAT-08, FEAT-09, FEAT-13, FEAT-25, FEAT-26 | FEAT-13 (the evidentiary record that makes immutability verifiable) |
| XBR-05 | Every record-worthy event — acceptance, change request, approval, reopen, invoice sent, deliverable upload/removal, a contact's first view of a proposal/deliverable/invoice, reminders, contact role changes, refunds/reversals/cancellations, manual payments, support sessions — writes an append-only trail entry with actor and timestamp. | FEAT-03, FEAT-05, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-13, FEAT-18, FEAT-25, FEAT-31 | FEAT-13 (owns the Activity Log Entry) |
| XBR-06 | Editing a sent-but-unaccepted proposal voids the prior version and re-sends; a project has at most one active proposal; a voided proposal cannot be accepted and the client is directed to the current one. | FEAT-02, FEAT-03 | FEAT-02 (owns proposal versions) |
| XBR-07 | A proposal cannot be sent until the client has at least one Primary contact, and the last Primary contact cannot be removed until a replacement is designated. | FEAT-02, FEAT-03, FEAT-18 | FEAT-18 (owns contact roles) |
| XBR-08 | Role entitlements follow the Access Matrix everywhere: only Primary contacts accept proposals, request changes, approve milestones, and see, pay, and download invoices; Reviewer contacts view and comment only and never see proposal or invoice content; notification recipients are limited to contacts entitled to the event. | FEAT-03, FEAT-05, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-14, FEAT-18, FEAT-25, FEAT-26 | FEAT-18 (assigns the Primary/Reviewer role) |
| XBR-09 | Client isolation: a contact reaches only their own company's projects under one freelancer; a person who is a contact for several freelancers sees each portal separately; an out-of-scope or expired link shows a plain explanation and a fresh-link option, never another company's data. | FEAT-05, FEAT-18, FEAT-03, FEAT-06, FEAT-07, FEAT-08, FEAT-10 | FEAT-05 (owns portal access and scoping) |
| XBR-10 | A milestone that has been approved or invoiced cannot be removed or re-priced; changes go through a logged reopen (FEAT-08) or an invoice correction (FEAT-09); schedule adjustments are dated and never retroactive. | FEAT-04, FEAT-08, FEAT-09 | FEAT-04 (owns milestones and the schedule) |
| XBR-11 | A deliverable on an approved milestone cannot be removed, only superseded by a new version; every removal is recorded in the trail. | FEAT-06, FEAT-08, FEAT-13, FEAT-17 | FEAT-06 (owns deliverables) |
| XBR-12 | Clients are notified about a deliverable only once its upload has fully completed, never for a partial file; interrupted uploads resume rather than restart. | FEAT-06, FEAT-14, FEAT-16 | FEAT-06 (owns the deliverable-ready event) |
| XBR-13 | Each re-upload creates a new immutable version; comments stay attached to the version they were made on; all versions count against the freelancer's storage allowance. | FEAT-06, FEAT-07, FEAT-16, FEAT-17 | FEAT-17 (owns Deliverable Version) |
| XBR-14 | Per-file size ceiling and per-freelancer storage allowance are enforced with a warning before the limit; the ceiling accommodates video somewhat over 1 GB. | FEAT-06, FEAT-16, FEAT-17, FEAT-23 | FEAT-16 (owns storage limits) |
| XBR-15 | Overdue reminders go out at day 3 and day 10 after the due date, counted in the freelancer's time zone; they stop the instant the invoice is paid, pause while a bank transfer is pending or when the freelancer pauses that invoice, and manual reminders are limited to one per invoice per day. | FEAT-09, FEAT-10, FEAT-11, FEAT-15 | FEAT-11 (owns the reminder schedule) |
| XBR-16 | Every invoice carries a due date from the freelancer's default payment terms (adjustable before sending), a unique sequential number per freelancer, her business details, and the client's billing details; sending is blocked until both sets of details exist. | FEAT-01, FEAT-09, FEAT-11, FEAT-21 | FEAT-09 (owns invoice content) |
| XBR-17 | A project's currency and tax line must be set before its first invoice, proposal prices are in that currency, the tax line is a freelancer-configured label and rate (no automatic calculation), and the currency cannot change once the first invoice is sent. | FEAT-02, FEAT-09, FEAT-15 | FEAT-15 (owns currency and tax configuration) |
| XBR-18 | Financial totals are shown per currency; amounts in different currencies are never converted or added together. | FEAT-12, FEAT-15, FEAT-22 | FEAT-15 (owns currency) |
| XBR-19 | With no connected payment account, invoices still issue and send with instructions for paying the freelancer directly; when the connection needs attention, pay links show "online payment is temporarily unavailable"; disconnecting warns that open invoices lose their pay links. | FEAT-09, FEAT-10, FEAT-20, FEAT-32 | FEAT-32 (owns the connection and pay-link readiness) |
| XBR-20 | An invoice is paid once and in full only; a manually recorded payment must be the full amount and not future-dated; a refund cannot exceed the amount paid; a refunded invoice cannot be marked Paid again without a logged correction. | FEAT-10, FEAT-25, FEAT-12, FEAT-22 | FEAT-10 (owns Payment) |
| XBR-21 | When the payment processor reports a chargeback or reversal on a paid invoice, the invoice shows Disputed alongside its original Paid record and the freelancer is notified; refunds and dispute responses happen in her own processor account, never inside Clientroom. | FEAT-10, FEAT-25, FEAT-32, FEAT-14 | FEAT-25 (owns refund, reversal, and cancellation status) |
| XBR-22 | Financial Dashboard and Accounting Export totals are derived only from Invoice and Payment records, including refunds, reversals, and manually recorded payments. | FEAT-09, FEAT-10, FEAT-12, FEAT-22, FEAT-25 | FEAT-10 (owns payment truth; FEAT-09 owns invoice truth) |
| XBR-23 | Adding or reactivating an active client beyond the free-tier limit requires an active paid plan; when a paid plan ends, no data is lost and existing portals stay reachable, but adding clients beyond the limit is blocked. | FEAT-01, FEAT-23 | FEAT-23 (owns the plan and its limits) |
| XBR-24 | Client deletion is allowed only while the client has no sent proposal, invoice, or activity; archiving a client or project with unpaid invoices or pending approvals requires explicit confirmation and never erases records. | FEAT-01, FEAT-02, FEAT-09, FEAT-13 | FEAT-01 (owns Client and Project) |
| XBR-25 | Marking a project cancelled preserves its full history; an unaccepted proposal stays open until the freelancer revises and re-sends it or cancels the project. | FEAT-01, FEAT-03, FEAT-25 | FEAT-25 (owns cancellation) |
| XBR-26 | A request-changes note from the Primary contact is recorded as a comment on the proposal, notifies the freelancer immediately, and never alters the proposal itself. | FEAT-02, FEAT-03, FEAT-07 | FEAT-03 (owns the change-request action) |
| XBR-27 | A contact's erasure request ends their access immediately and removes their contact details, while acceptances and approvals they gave remain on the record under their name. | FEAT-03, FEAT-08, FEAT-13, FEAT-18 | FEAT-18 (owns contact removal) |
| XBR-28 | Magic links are single-use and time-limited; requesting a new link invalidates earlier unused ones; only contacts added through contact management are recognized. | FEAT-05, FEAT-18 | FEAT-05 (owns sign-in) |
| XBR-29 | The operator's support sessions are read-only in every feature, cover one account at a time, end after inactivity, exclude file downloads and data/accounting exports, are always announced to the freelancer by email, and are always listed in her trail. | FEAT-31, FEAT-13, FEAT-14, FEAT-06, FEAT-16, FEAT-22, FEAT-24 | FEAT-31 (owns support access) |
| XBR-30 | Notification preferences can switch off only optional emails; transactional emails core to the record (e.g., payment confirmations) always send; delivery failures are surfaced to the freelancer as warnings on the affected project. | FEAT-14, FEAT-21 | FEAT-14 (owns email delivery) |
| XBR-31 | The freelancer's logo and brand colour apply to every client-facing screen and email, fall back to a clean neutral default, and are adjusted for legibility; the referral mark sits alongside the branding without overriding it. | FEAT-02, FEAT-05, FEAT-06, FEAT-09, FEAT-14, FEAT-19, FEAT-33 | FEAT-19 (owns the Branding Profile) |
| XBR-32 | The referral mark appears on every client-facing page and email on every plan in MVP, never reveals client, project, or freelancer data, and attribution is used only in aggregate; the "how did you hear" answer is asked during onboarding. | FEAT-05, FEAT-14, FEAT-20, FEAT-33 | FEAT-33 (owns Referral Attribution) |
| XBR-33 | Account deletion warns about open items (unpaid invoices, pending approvals), disconnects the payment account, removes all the freelancer's data including client contacts' personal data, and keeps only financial records under a legal retention requirement. | FEAT-24, FEAT-32, FEAT-01, FEAT-09, FEAT-10, FEAT-18 | FEAT-24 (owns account deletion) |
| XBR-34 | When enabled for a proposal (v1), a legally binding e-signature replaces the plain Accept click with the same Primary-only access and the same immutability; otherwise the timestamped Accept is the default. | FEAT-03, FEAT-26 | FEAT-03 (owns the acceptance record; FEAT-26 extends it) |
| XBR-35 | With a verified custom domain (Later), the portal and client-facing links use it, and the shared default address always remains available as a fallback. | FEAT-05, FEAT-14, FEAT-27 | FEAT-27 (owns the Custom Domain Record) |

## External Touchpoints

Category-level external capabilities from the `## Dependencies` section of assumptions-constraints.md (ASMP-28 to ASMP-32), plus capability categories discovered by validated Feature Breakdown Briefs (Step 7.5 rule 4), mapped to the features that rely on them. The Integration Specs column was filled incrementally as analysis batches validated their Feature Breakdown Briefs. The full coverage check ran at the final analysis batch (FEAT-30) across all 33 validated Briefs and passed in both directions: every capability row has at least one covering Integration spec, and every Integration spec in the Briefs (7 in total) is mapped to a row.

| Capability Category | Features Involved | Integration Specs |
|---------------------|-------------------|-------------------|
| Payment processing into each freelancer's own account — connection, card and bank-transfer payment, status, pending transfers, reversals (ASMP-28) | FEAT-09, FEAT-10, FEAT-20, FEAT-25, FEAT-32 | FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing — payment submission, pending bank transfers, outcome reporting). FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting — connect/reconnect hand-off, readiness status, attention reason, available payment methods, and inbound reversal/chargeback notices relayed to FEAT-25 per XBR-21; authority for pay-link readiness per XBR-19). FEAT-25 validated with no Integration spec for this capability: it records reversals by consuming the inbound reversal/chargeback notice relayed by FEAT-32's integration (FEAT-25.SPEC-005 → FEAT-32.SPEC-002, XBR-21). FEAT-09 validated with no Integration spec for this capability: it derives pay-link availability from the connection status (FEAT-09.SPEC-009 → FEAT-32, XBR-19). FEAT-20 validated with no Integration spec for this capability: its optional Connect payments step only navigates into FEAT-32's connection flow (FEAT-20.SPEC-002 → FEAT-32.SPEC-002) |
| Transactional email delivery with delivery and bounce status (ASMP-29) | FEAT-14 (delivery), relied on by FEAT-02, FEAT-03, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-20, FEAT-21, FEAT-23, FEAT-24, FEAT-25, FEAT-26, FEAT-27, FEAT-31, FEAT-32 | FEAT-14.SPEC-001 (Transactional Email Delivery — sends every composed email and reports delivery, bounce, and failure status back). FEAT-02 and FEAT-03 validated with no Integration spec for this capability: they use it through Notification specs FEAT-02.SPEC-011, FEAT-03.SPEC-006, and FEAT-03.SPEC-007. FEAT-05, FEAT-06, FEAT-07, and FEAT-08 validated with no Integration spec for this capability: they use it through Notification specs FEAT-05.SPEC-008, FEAT-06.SPEC-006, FEAT-07.SPEC-003, FEAT-07.SPEC-004, and FEAT-08.SPEC-007. FEAT-09, FEAT-10, and FEAT-11 validated with no Integration spec for this capability: they use it through Notification specs FEAT-09.SPEC-010, FEAT-10.SPEC-007, and FEAT-11.SPEC-004. FEAT-18 and FEAT-32 validated with no Integration spec for this capability: they use it through Notification specs FEAT-18.SPEC-010, FEAT-18.SPEC-011, and FEAT-32.SPEC-006. FEAT-20, FEAT-21, and FEAT-23 validated with no Integration spec for this capability: they use it through Notification specs FEAT-20.SPEC-006, FEAT-21.SPEC-011, and FEAT-23.SPEC-008 (FEAT-21.SPEC-005 also sends its re-verification link through it). FEAT-24, FEAT-25, and FEAT-31 validated with no Integration spec for this capability: they use it through Notification specs FEAT-24.SPEC-008, FEAT-24.SPEC-009, FEAT-25.SPEC-007, FEAT-25.SPEC-008, FEAT-31.SPEC-006, and FEAT-31.SPEC-007. FEAT-26 and FEAT-27 validated with no Integration spec for this capability: they use it through Notification specs FEAT-26.SPEC-004 (signed-copy confirmation to both parties) and FEAT-27.SPEC-004 (custom domain verified confirmation) |
| Large-file storage and delivery with version history (ASMP-30) | FEAT-06, FEAT-16, FEAT-17 | FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability — resumable ingestion, streaming/download delivery, reported transfer status, within the stated budget; authority for storage limits per XBR-14). FEAT-17 validated with no Integration spec for this capability: it consumes storage through FEAT-16 (FEAT-17.SPEC-003 → FEAT-16.SPEC-007, XBR-13, XBR-14), mirroring FEAT-06. FEAT-06 validated with no Integration spec for this capability: it consumes storage through FEAT-16 (FEAT-06.SPEC-003 → FEAT-16, XBR-12, XBR-14) |
| Subscription billing for the freelancer's own plan (ASMP-31) | FEAT-23 | FEAT-23.SPEC-003 (Subscription Billing Processing — submits upgrade, downgrade, and cancellation changes to the subscription-billing capability and receives charge outcomes, renewal and period-end events, and failure reasons; FEAT-23.SPEC-004 applies them to the Subscription Plan) |
| Domain verification and secure serving at a freelancer's own domain — Later phase (ASMP-32) | FEAT-27, FEAT-05 | FEAT-27.SPEC-002 (Domain Verification & Secure Serving — verifies Nadia controls the added domain, serves her portal securely at it once verified, and reports verification, failure reason, and re-check results; FEAT-27.SPEC-003 guarantees the shared default address remains the fallback per XBR-35). FEAT-05 validated with no Integration spec for this capability: it only reads the Custom Domain Record (FEAT-05.SPEC-002, FEAT-05.SPEC-003; Later phase) |
| Electronic-signature attestation for legally binding proposal signing — v1 phase (not in the Dependencies section of assumptions-constraints.md; added from the validated FEAT-26 Brief per Step 7.5 rule 4) | FEAT-26 | FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability — submits the signature data captured by FEAT-26.SPEC-002 for attestation and returns the confirmation that gives the signed acceptance legal weight beyond a self-recorded timestamp; the jurisdictional standard it must meet is left to Stage 4) |
