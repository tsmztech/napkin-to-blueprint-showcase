---
document_type: product-features
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# Product Features

## Summary

This product includes 30 features: 16 Core, 9 Important, 5 Nice-to-Have. By phase: 23 MVP, 3 v1, 4 Later. By type: 18 User-Facing, 6 Platform, 6 Lifecycle. The product manages 17 domain entities. Core features close the full proposal-to-paid loop (proposal, milestones, deliverables, approval, invoicing, payment, reminders, dashboard, audit trail) plus the platform capabilities the loop cannot work without (portal access, notifications, currency/tax, large-file handling). Important features cover the surrounding correctness and business needs (versioning, contact roles, branding, onboarding, settings, accounting export, subscription billing, GDPR export/deletion, refunds). Nice-to-Have features are genuine value-adds the product can grow into without restructuring anything already built.

## Domain Entity Inventory

### Entity: Client
- **Description:** A client company the freelancer works with, the top-level container for its projects and contacts
- **Lifecycle:** Created -> Active -> Archived
- **Created by:** Client & Project Management (FEAT-01)
- **Managed by:** Client & Project Management (FEAT-01)
- **Referenced by:** Proposal Creation & Sending (FEAT-02), Client Contact Management & Roles (FEAT-18), Financial Dashboard (FEAT-12)

### Entity: Client Contact
- **Description:** A named person at a client company, tagged Primary or Reviewer, who authenticates via magic link
- **Lifecycle:** Invited -> Active -> Removed
- **Created by:** Client Contact Management & Roles (FEAT-18)
- **Managed by:** Client Contact Management & Roles (FEAT-18)
- **Referenced by:** Client Portal Access (FEAT-05), Proposal Acceptance (FEAT-03), Milestone Approval (FEAT-08), Deliverable Review & Feedback (FEAT-07), Invoice Payment Processing (FEAT-10)

### Entity: Project
- **Description:** A unit of work for a client, containing its own proposal, milestones, deliverables, and invoices
- **Lifecycle:** Draft -> In Progress -> Complete -> (optionally) Cancelled -> Archived
- **Created by:** Client & Project Management (FEAT-01)
- **Managed by:** Client & Project Management (FEAT-01), Refund & Cancelled Project Handling (FEAT-25)
- **Referenced by:** Proposal Creation & Sending (FEAT-02), Milestone & Payment Schedule Setup (FEAT-04), Financial Dashboard (FEAT-12), Immutable Activity & Audit Trail (FEAT-13)

### Entity: Proposal
- **Description:** The scope and price document sent to a client for acceptance
- **Lifecycle:** Draft -> Sent -> Accepted (or Voided by re-edit)
- **Created by:** Proposal Creation & Sending (FEAT-02)
- **Managed by:** Proposal Creation & Sending (FEAT-02), Proposal Acceptance (FEAT-03), Legally Binding E-Signature for Proposals (FEAT-26)
- **Referenced by:** Milestone & Payment Schedule Setup (FEAT-04), Immutable Activity & Audit Trail (FEAT-13)

### Entity: Milestone
- **Description:** A discrete stage of a project's work, each with its own price or trigger and its own approval
- **Lifecycle:** Defined -> Deliverable Uploaded -> Approved -> (optionally) Reopened
- **Created by:** Milestone & Payment Schedule Setup (FEAT-04)
- **Managed by:** Milestone & Payment Schedule Setup (FEAT-04), Milestone Approval (FEAT-08)
- **Referenced by:** Deliverable Upload & Sharing (FEAT-06), Invoice Generation & Sending (FEAT-09), Immutable Activity & Audit Trail (FEAT-13)

### Entity: Payment Schedule
- **Description:** The project-level configuration of how and when the client is billed (deposit, per-milestone, on completion, or a mix)
- **Lifecycle:** Set at proposal acceptance -> Adjustable during the project
- **Created by:** Milestone & Payment Schedule Setup (FEAT-04)
- **Managed by:** Milestone & Payment Schedule Setup (FEAT-04)
- **Referenced by:** Invoice Generation & Sending (FEAT-09)

### Entity: Deliverable
- **Description:** An uploaded file or linked external asset (Figma, Drive, Dropbox) the client reviews and comments on
- **Lifecycle:** Uploaded/Linked -> Active -> (optionally) Superseded by a new version
- **Created by:** Deliverable Upload & Sharing (FEAT-06)
- **Managed by:** Deliverable Upload & Sharing (FEAT-06), Large File Handling & Storage (FEAT-16)
- **Referenced by:** Deliverable Review & Feedback (FEAT-07), Milestone Approval (FEAT-08), Deliverable Version History (FEAT-17)

### Entity: Deliverable Version
- **Description:** A specific uploaded round of a deliverable, preserved rather than overwritten
- **Lifecycle:** Created -> Superseded (never deleted)
- **Created by:** Deliverable Upload & Sharing (FEAT-06), Deliverable Version History (FEAT-17)
- **Managed by:** Deliverable Version History (FEAT-17)
- **Referenced by:** Deliverable Review & Feedback (FEAT-07)

### Entity: Comment
- **Description:** Feedback text pinned to a specific deliverable (version) by a client contact or the freelancer
- **Lifecycle:** Posted -> (optionally) Retracted
- **Created by:** Deliverable Review & Feedback (FEAT-07)
- **Managed by:** Deliverable Review & Feedback (FEAT-07)
- **Referenced by:** Milestone Approval (FEAT-08)

### Entity: Invoice
- **Description:** A billing document tied to a deposit, milestone approval, or project completion, sent to the client's Primary Contact
- **Lifecycle:** Generated -> Sent -> Paid (or Overdue -> Paid, or Refunded, or Corrected via a new invoice)
- **Created by:** Invoice Generation & Sending (FEAT-09)
- **Managed by:** Invoice Generation & Sending (FEAT-09), Invoice Payment Processing (FEAT-10), Automated Payment Reminders (FEAT-11), Refund & Cancelled Project Handling (FEAT-25)
- **Referenced by:** Financial Dashboard (FEAT-12), Currency & Tax Handling (FEAT-15), Accounting Export (FEAT-22), Immutable Activity & Audit Trail (FEAT-13)

### Entity: Payment
- **Description:** A confirmed payment transaction against an invoice, processed directly into the freelancer's own processor account
- **Lifecycle:** Initiated -> Succeeded (or Failed)
- **Created by:** Invoice Payment Processing (FEAT-10)
- **Managed by:** Invoice Payment Processing (FEAT-10)
- **Referenced by:** Financial Dashboard (FEAT-12), Accounting Export (FEAT-22)

### Entity: Reminder Log
- **Description:** The record of automated overdue-payment reminders sent for a given invoice
- **Lifecycle:** Scheduled -> Sent -> (optionally) Paused
- **Created by:** Automated Payment Reminders (FEAT-11)
- **Managed by:** Automated Payment Reminders (FEAT-11)
- **Referenced by:** Immutable Activity & Audit Trail (FEAT-13)

### Entity: Activity Log Entry
- **Description:** An append-only record of a record-worthy event (acceptance, approval, invoice sent) — the evidence trail
- **Lifecycle:** Written -> Permanent (never edited or deleted)
- **Created by:** Immutable Activity & Audit Trail (FEAT-13)
- **Managed by:** N/A — append-only; no feature edits existing entries
- **Referenced by:** In-App Notification Center (FEAT-29)

### Entity: Branding Profile
- **Description:** The freelancer's logo and brand colour applied to every client-facing surface
- **Lifecycle:** Unset (default) -> Configured -> Updated
- **Created by:** Freelancer Branding (FEAT-19)
- **Managed by:** Freelancer Branding (FEAT-19)
- **Referenced by:** Client Portal Access (FEAT-05), Proposal Creation & Sending (FEAT-02), Invoice Generation & Sending (FEAT-09)

### Entity: Subscription Plan
- **Description:** The freelancer's own account-level plan and billing state for using Clientroom itself
- **Lifecycle:** Free -> Paid (upgrade) -> (optionally) Downgraded
- **Created by:** Subscription Plan & Billing Management (FEAT-23)
- **Managed by:** Subscription Plan & Billing Management (FEAT-23)
- **Referenced by:** Client & Project Management (FEAT-01, client-count limit check)

### Entity: Accounting Export File
- **Description:** A generated CSV or QuickBooks/Xero-compatible export of a freelancer's invoices and payments
- **Lifecycle:** Generated -> Downloaded
- **Created by:** Accounting Export (FEAT-22)
- **Managed by:** N/A — a point-in-time generated file, not an ongoing managed record
- **Referenced by:** N/A — consumed outside the product in the freelancer's own accounting software

### Entity: Custom Domain Record
- **Description:** A freelancer's own domain configured and verified to serve their portal
- **Lifecycle:** Added -> Verifying -> Verified (or Verification Failed)
- **Created by:** Custom Domain per Freelancer (FEAT-27)
- **Managed by:** Custom Domain per Freelancer (FEAT-27)
- **Referenced by:** Client Portal Access (FEAT-05)

## Core Features

### Client & Project Management

**ID:** FEAT-01

**Description:** The freelancer can add client companies, create projects under each one, and see every client and project she manages in one place, each showing its current stage.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief's whole premise is one place per client project (BRIEF.md, Vision); nothing else in the product has anywhere to attach without this container. MVP phase: every other feature depends on a client and project existing first.

**Connected Entities:** Client (create, read, update, archive), Project (create, read, update, archive)

**Key Capabilities:**
- Add a client company — freelancer records a new client she works with
- Create a project under a client — a unit of work with its own proposal, milestones, and invoices
- View all clients and projects — a single roster with each project's current stage
- Archive a client or project — remove it from active view without deleting its history

**Primary Flows & Alternates:**
- Happy path: add client -> create project -> project appears in the roster at stage "Draft," ready for a proposal
- Archiving with open items: archiving a client or project with unpaid invoices or pending approvals requires an explicit confirmation, since it does not erase the record
- Renaming: freelancer renames a client or project after creation without breaking any existing links to it

**States:** Empty: no clients yet shows a prompt to add the first one. Loading: the roster renders with skeleton rows while loading a large client list. Error: a failed save preserves entered fields and offers retry. Offline-degraded: composing a new client/project offline is not supported; a clear connectivity notice is shown and the action retries once online.

**Validation & Limits:** Client name and project name are required; a project belongs to exactly one client; no hard cap on client count (BRIEF.md, Scale: 3–15 active clients per freelancer, a few thousand freelancers in year one).

**Access:** Nadia (Freelancer) has Full access, per the Access Matrix in user-persona.md. Client contacts have no access to this management surface — they see only their own company's project state through their scoped portal view (FEAT-05).

**Communications:** N/A — this is the freelancer's private workspace; adding a client or project triggers no client-facing message.

**Data Notes:** Captured: client name, project name, basic contact reference. Displayed: roster with computed project stage. Derived: a project's "stage" label is derived from its proposal/milestone/invoice state elsewhere. Source: freelancer input.

**Interactions:** Feeds Proposal Creation & Sending (FEAT-02), Milestone & Payment Schedule Setup (FEAT-04), and Financial Dashboard (FEAT-12); read by Subscription Plan & Billing Management (FEAT-23) for client-count limits.

**Signals:** client_created, project_created, project_archived, client_archived.

### Proposal Creation & Sending

**ID:** FEAT-02

**Description:** The freelancer drafts a proposal with scope and price for a project and sends it to the client's Primary Contact as a link, replacing the "PDF in email" workflow described in the brief.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Experience narrative opens with "You send a proposal to a new client from Clientroom" — this is the product's entry point into every project. MVP phase: the core loop cannot start without it.

**Connected Entities:** Proposal (create, update, send)

**Key Capabilities:**
- Draft a proposal — freelancer writes scope and sets a price for the project
- Send the proposal — client's Primary Contact receives a link by email
- Edit before acceptance — freelancer can revise a sent-but-unaccepted proposal

**Primary Flows & Alternates:**
- Happy path: draft the proposal -> send it -> the client receives the link and can review it
- Edit after sending: editing a sent proposal before it is accepted voids the prior version and re-sends the updated one, so the client is never shown an outdated price
- Resend: freelancer resends the link if the client cannot find the original email

**States:** Empty: a project with no proposal shows a prompt to draft one. Loading: N/A — the draft form is local until sent. Error: a failed send preserves the draft and offers retry. Offline-degraded: drafting can continue with unsaved local state; sending requires connectivity.

**Validation & Limits:** Scope description and price are required; price must be a positive amount in the project's set currency (FEAT-15); a project has at most one active (non-voided) proposal at a time.

**Access:** Nadia has Full access. Owen (Client Primary Contact) can view and accept the proposal he receives (Own-only, via FEAT-03) but cannot create or edit one. Priya (Client Reviewer Contact) has no access to proposal content, per the Access Matrix.

**Communications:** Sends an email to the client's Primary Contact with the proposal link when a proposal is sent or resent.

**Data Notes:** Captured: scope description, price, currency, and a reference to the project's payment schedule. Displayed: a proposal preview before sending. Derived: none. Source: freelancer input, plus Milestone & Payment Schedule Setup (FEAT-04) once drafted.

**Interactions:** Feeds Proposal Acceptance (FEAT-03); depends on Client & Project Management (FEAT-01) for the owning project and Currency & Tax Handling (FEAT-15) for currency.

**Signals:** proposal_drafted, proposal_sent, proposal_edited_before_acceptance, proposal_resent.

### Proposal Acceptance

**ID:** FEAT-03

**Description:** The client's Primary Contact reviews a sent proposal and accepts it with one click; the acceptance is recorded with a timestamp that can never be silently altered, and a deposit invoice appears immediately if the payment schedule calls for one.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Experience narrative: "clicks 'Accept.' The acceptance is recorded with a timestamp, a deposit invoice appears" — this is the moment a project becomes real and billable. MVP phase: required for the deposit-invoice trigger and the evidence record the brief calls out as a Success Criterion.

**Connected Entities:** Proposal (update: accepted state), Invoice (create: deposit invoice)

**Key Capabilities:**
- Review the proposal — client reads scope and price before deciding
- Accept with one click — records a timestamped, permanent acceptance
- Automatic deposit invoicing — a deposit invoice is generated immediately when the schedule includes one

**Primary Flows & Alternates:**
- Happy path: Owen opens the link, reads scope and price, clicks Accept, the timestamp is recorded, and a deposit invoice appears if applicable
- Not ready to accept: Owen simply does not click Accept; there is no formal decline — Nadia follows up outside the product to revise and resend if needed
- Outdated proposal: if the proposal was edited after being sent, the prior version is shown as voided and the client is directed to the current one

**States:** Empty: N/A — this view exists only once a proposal has been sent. Loading: proposal content renders fully before the Accept control becomes active, preventing an accidental early tap. Error: a failed accept action can be retried without recording a duplicate or losing the click's intent. Offline-degraded: acceptance requires connectivity, since it is a timestamped record; the client sees a "reconnect to accept" message rather than a false success.

**Validation & Limits:** A proposal can be accepted exactly once; a voided proposal cannot be accepted.

**Access:** Owen (Primary Contact) can accept (Own-only). Priya (Reviewer) sees the proposal read-only with no Accept control, per the Access Matrix. Nadia sees the resulting status but does not perform the acceptance herself.

**Communications:** Confirmation email to Owen and Nadia when acceptance is recorded; the auto-generated deposit invoice sends its own notification (FEAT-09).

**Data Notes:** Captured: acceptance timestamp and the accepting contact's identity. Displayed: an "Accepted on {date}" marker thereafter. Derived: none. Source: the accept action itself, written once and never silently altered (BRIEF.md, Constraints: record immutability).

**Interactions:** Depends on Proposal Creation & Sending (FEAT-02); triggers Invoice Generation & Sending (FEAT-09) for the deposit; feeds Immutable Activity & Audit Trail (FEAT-13).

**Signals:** proposal_viewed_by_client, proposal_accepted, deposit_invoice_auto_generated.

### Milestone & Payment Schedule Setup

**ID:** FEAT-04

**Description:** The freelancer defines the project's milestones and chooses how the client is billed — deposit, per-milestone, on completion, or a mix — so invoicing can be automatic later in the project.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Target Users & Roles: the freelancer "sets milestones and payment schedules (deposit, per milestone, on completion, or a mix)." Without this, invoices have no trigger to fire from. MVP phase: required before any milestone-based invoice can exist.

**Connected Entities:** Milestone (create, update), Payment Schedule (create, update)

**Key Capabilities:**
- Define milestones — name and order the project's discrete stages of work
- Set a payment structure — choose deposit, per-milestone, on completion, or a mix
- Adjust the schedule — change milestones or pricing mid-project as scope evolves

**Primary Flows & Alternates:**
- Happy path: define milestones with a price or trigger each -> schedule attaches to the accepted proposal and drives invoicing automatically
- Mid-project change: freelancer adjusts milestones or pricing after the project has started; the change is dated, not retroactive
- Price mismatch: the sum of milestone prices does not match the proposal total — the freelancer is flagged, not blocked, since scope can legitimately change

**States:** Empty: a project with no milestones shows a prompt to add the first one. Loading: N/A — local editing. Error: a failed save preserves entered milestone data. Offline-degraded: editing can continue locally and syncs once connectivity returns.

**Validation & Limits:** Each milestone requires a name and either a price or a "no separate charge" flag; at least one payment trigger must exist for the project to invoice at all.

**Access:** Nadia has Full access. Owen and Priya see the resulting milestone list read-only within their project view (Own-only, per the Access Matrix); neither edits the schedule.

**Communications:** N/A — the schedule itself sends no message; invoices it later triggers send their own (FEAT-09).

**Data Notes:** Captured: milestone names, prices, trigger types, target dates. Displayed: the project's milestone timeline. Derived: none. Source: freelancer input, informed by the accepted proposal.

**Interactions:** Depends on Proposal Acceptance (FEAT-03); feeds Milestone Approval (FEAT-08) and Invoice Generation & Sending (FEAT-09).

**Signals:** milestone_created, milestone_schedule_edited, payment_trigger_set.

### Client Portal Access (Magic-Link Login)

**ID:** FEAT-05

**Description:** A client contact signs in passwordless, by requesting a one-time email link, and lands in a view scoped strictly to their own company's projects — no account-creation step, no password to remember.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md, Target Users & Roles: contacts "sign in passwordless with a magic link by email and never hit an account-creation wall." Without this, no client can reach any client-facing feature at all. MVP phase: every client-facing feature depends on it.

**Connected Entities:** Client Contact (authenticate)

**Key Capabilities:**
- Request a sign-in link — contact enters their email and receives a one-time link
- Enter the portal — clicking the link opens a view scoped to their own company only
- Re-request access — a fresh link can be requested any time, invalidating unused prior ones

**Primary Flows & Alternates:**
- Happy path: contact clicks a link from an email (new proposal, deliverable, approval request, or invoice) or requests one directly, and lands in their scoped portal
- Expired or reused link: the contact sees a clear explanation and can request a fresh link in one step
- No projects yet: a contact with no active project sees a plain "nothing here yet" message, not an error

**States:** Empty: a contact with no projects yet (before any proposal is sent) sees a plain message rather than an error. Loading: link verification shows a brief in-progress state before landing in the portal. Error: an expired or invalid link shows a clear explanation and a one-tap way to request a new one. Offline-degraded: the sign-in link cannot be verified offline; the contact sees a connectivity notice.

**Validation & Limits:** A magic link is single-use and time-limited; requesting a new link invalidates prior unused ones for that contact.

**Access:** Any recognized Client Contact (Owen or Priya) can request access to their own company's scope only, per the Access Matrix; a contact can never see another client company's data (BRIEF.md, Constraints: strict isolation between clients).

**Communications:** Sends the sign-in link by email on every login request.

**Data Notes:** Captured: contact email and login timestamps. Displayed: none beyond the sign-in state. Derived: none. Source: contact-provided email, matched against contacts the freelancer has added (FEAT-18).

**Interactions:** Gates access to Proposal Acceptance (FEAT-03), Deliverable Upload & Sharing (FEAT-06), Deliverable Review & Feedback (FEAT-07), Milestone Approval (FEAT-08), Invoice Payment Processing (FEAT-10); depends on Client Contact Management & Roles (FEAT-18) for which contacts are recognized.

**Signals:** magic_link_requested, magic_link_used, magic_link_expired.

### Deliverable Upload & Sharing

**ID:** FEAT-06

**Description:** The freelancer uploads a file — a design file, video, or PDF, up to large sizes — or attaches a link to Figma, Google Drive, or Dropbox, and attaches it to a milestone; the client is notified it is ready to review.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Problem Statement centers on files scattered across "final_v3_REAL" versions in a shared drive; this feature is the direct fix. MVP phase: without it, there is nothing for the client to approve.

**Connected Entities:** Deliverable (create, update), Milestone (read)

**Key Capabilities:**
- Upload a file — design file, video, or PDF, resumable for large sizes
- Link an external asset — attach a Figma, Drive, or Dropbox link instead of copying the file in
- Notify the client — the relevant contacts are emailed when a deliverable is ready

**Primary Flows & Alternates:**
- Happy path: choose a milestone -> upload a file or paste a link -> the deliverable appears in the client's portal and an email goes out
- Interrupted upload: a large upload interrupted by a dropped connection resumes rather than restarting from zero
- Linked asset unreachable: a link that fails to resolve is flagged to the freelancer before it is shown to the client as broken

**States:** Empty: a milestone with no deliverables yet shows a prompt to upload the first one. Loading: upload shows real progress for large files, not an indefinite spinner. Error: a failed upload preserves the selected file and offers retry without re-selecting it. Offline-degraded: an in-progress upload pauses and resumes automatically when connectivity returns.

**Validation & Limits:** Linked deliverables require a valid, reachable link, per BRIEF.md's Ecosystem section (accepted by link, not copied in, for v1); uploaded files are capped by a size ceiling consistent with the roughly $100/month infrastructure constraint (BRIEF.md, Constraints).

**Access:** Nadia has Full access (upload, replace, remove). Owen and Priya have view-only access to deliverables on their own company's projects (Own-only, per the Access Matrix); neither uploads or deletes.

**Communications:** Email to relevant client contacts ("a new deliverable is ready to review") when a deliverable is uploaded or linked.

**Data Notes:** Captured: file or link, upload timestamp, owning milestone. Displayed: deliverable list per milestone with preview where feasible. Derived: none. Source: freelancer upload, or a linked third-party tool referenced but not copied in.

**Interactions:** Depends on Milestone & Payment Schedule Setup (FEAT-04); feeds Deliverable Review & Feedback (FEAT-07), Milestone Approval (FEAT-08), Deliverable Version History (FEAT-17), and Large File Handling & Storage (FEAT-16).

**Signals:** deliverable_uploaded, deliverable_linked, upload_resumed, upload_failed.

### Deliverable Review & Feedback

**ID:** FEAT-07

**Description:** Client contacts leave comments pinned to a specific deliverable; the freelancer sees all feedback in one thread per deliverable, replacing scattered WhatsApp screenshots.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Experience narrative: the client "leaves three comments pinned to the files," and the Problem Statement names WhatsApp screenshots as the exact failure this replaces. MVP phase: needed before a client can meaningfully decide to approve.

**Connected Entities:** Comment (create, read), Deliverable (read)

**Key Capabilities:**
- Leave a pinned comment — a comment attached to a specific deliverable
- See a full feedback thread — freelancer sees every comment from every authorized contact in one place
- Retract a comment — a contact can withdraw their own comment

**Primary Flows & Alternates:**
- Happy path: contact opens a deliverable, leaves a comment pinned to it, freelancer is notified and replies in the same thread
- Multiple commenters: several contacts from the same company comment on the same deliverable; all are visible to each other and to Nadia
- Retraction: a contact retracts a comment (soft-removed, not silently rewritten, preserving the record's reliability)

**States:** Empty: a deliverable with no comments shows a plain "no feedback yet" state. Loading: the comment thread loads with a lightweight in-progress indicator on slow connections. Error: a failed comment submission preserves the typed text and offers retry. Offline-degraded: a comment composed offline is held locally and sent once connectivity returns.

**Validation & Limits:** A comment requires non-empty text (1–2,000 characters); comments cannot be silently edited after a short grace window, but can be retracted.

**Access:** Both Owen and Priya can comment (Own-only) on their own company's deliverables, per the Access Matrix; Nadia has Full visibility and can reply; contacts from other client companies never see this thread.

**Communications:** Email to Nadia when a client comments; email to the client contact when Nadia replies.

**Data Notes:** Captured: comment text, author, timestamp, pinned deliverable. Displayed: threaded per deliverable. Derived: none. Source: contact and freelancer input.

**Interactions:** Depends on Deliverable Upload & Sharing (FEAT-06); feeds Milestone Approval (FEAT-08) as context for the approval decision.

**Signals:** comment_posted, comment_retracted, comment_notification_sent.

### Milestone Approval

**ID:** FEAT-08

**Description:** The client's Primary Contact approves a milestone once satisfied; approval is timestamped and permanent, and approving automatically issues the next invoice per the payment schedule.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Experience narrative: "the founder hits 'Approve,' and approving the milestone automatically issues the next invoice" — this is the product's central automation and the record cited as evidence in scope disputes (BRIEF.md, Success Criteria). MVP phase: the core billing loop depends on it.

**Connected Entities:** Milestone (update: approved state), Invoice (create: next invoice)

**Key Capabilities:**
- Approve a milestone — a timestamped, permanent decision
- Automatic next invoice — approval triggers the next invoice in the payment schedule
- Reopen (freelancer only) — a logged, non-silent way to reopen a milestone if needed

**Primary Flows & Alternates:**
- Happy path: Owen reviews the deliverable and any feedback, clicks Approve, the timestamp is recorded, and the next invoice generates automatically
- Not yet satisfied: Owen leaves feedback via FEAT-07 instead of approving; the milestone stays open
- Reopen: Nadia reopens an approved milestone if genuinely necessary; this is itself a logged event, never a silent edit to the original approval

**States:** Empty: N/A — this view exists only once a deliverable has been uploaded for the milestone. Loading: the Approve control stays disabled until the deliverable and comment thread have fully loaded, preventing a premature approval. Error: a failed approval action is retried without double-recording. Offline-degraded: approval requires connectivity, shown as a clear "reconnect to approve" state.

**Validation & Limits:** A milestone can be approved exactly once; approval cannot be reversed by the client — only Nadia can reopen a milestone, logged as a distinct event.

**Access:** Owen (Primary Contact) can approve (Own-only). Priya (Reviewer) sees the milestone's status but no Approve control, per the Access Matrix.

**Communications:** Confirmation email to Owen and Nadia when approval is recorded; the auto-issued invoice sends its own notification (FEAT-09).

**Data Notes:** Captured: approval timestamp and the approving contact's identity. Displayed: an "Approved on {date}" marker thereafter. Derived: none. Source: the approval action, written once and never silently altered (BRIEF.md, Constraints: record immutability).

**Interactions:** Depends on Deliverable Upload & Sharing (FEAT-06) and Deliverable Review & Feedback (FEAT-07); triggers Invoice Generation & Sending (FEAT-09); feeds Immutable Activity & Audit Trail (FEAT-13).

**Signals:** milestone_approved, milestone_reopened_by_freelancer, milestone_invoice_auto_generated.

### Invoice Generation & Sending

**ID:** FEAT-09

**Description:** An invoice is generated whenever a deposit is accepted, a milestone is approved, or a project is marked complete, and sent to the client's Primary Contact by email with a pay link.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Problem Statement: invoices today "live in a spreadsheet plus a pasted payment link that gets chased for weeks" — automatic, correctly-triggered invoicing is a primary reason the product exists. MVP phase: without it, approvals and deposits have no financial consequence.

**Connected Entities:** Invoice (create, send)

**Key Capabilities:**
- Automatic invoicing — a deposit, milestone approval, or project completion triggers the right invoice
- Ad-hoc invoicing — freelancer can issue an invoice manually outside the schedule
- Send with a pay link — the client receives the invoice with a direct way to pay

**Primary Flows & Alternates:**
- Happy path: a payment trigger fires -> an invoice is generated with the correct amount, currency, and tax line -> it is emailed to Owen
- Ad-hoc invoice: Nadia issues an invoice outside the schedule (e.g., a scope addition)
- Correction: a sent invoice is never silently edited; a correction is a visible credit note or a new invoice, per BRIEF.md's record-immutability constraint

**States:** Empty: a project with no invoices yet shows "no invoices issued." Loading: N/A — generation is near-instant. Error: a failed send is retried without creating a duplicate invoice. Offline-degraded: N/A — invoice generation is a server-side trigger; if Nadia is offline when a trigger fires, the invoice still issues once processed.

**Validation & Limits:** Amount must match the triggering milestone/deposit/completion price plus tax; a sent invoice cannot be silently edited.

**Access:** Nadia has Full visibility and can issue ad-hoc invoices. Owen (Own-only) can view and pay invoices for his company. Priya has no invoice access, per the Access Matrix.

**Communications:** Email to Owen with the invoice and pay link when sent; a copy confirmation to Nadia.

**Data Notes:** Captured: amount, currency, tax line, triggering event. Displayed: invoice detail and status (sent, paid, overdue) in both the dashboard and the client portal. Derived: the tax line, per Currency & Tax Handling (FEAT-15). Source: the triggering milestone/proposal/deposit event.

**Interactions:** Triggered by Proposal Acceptance (FEAT-03) and Milestone Approval (FEAT-08); depends on Currency & Tax Handling (FEAT-15); feeds Invoice Payment Processing (FEAT-10), Automated Payment Reminders (FEAT-11), and Financial Dashboard (FEAT-12).

**Signals:** invoice_generated, invoice_sent, invoice_manually_issued, invoice_correction_issued.

### Invoice Payment Processing

**ID:** FEAT-10

**Description:** The client pays an invoice by card or bank transfer directly from the portal; the payment lands in the freelancer's own processor account, and the invoice's status updates immediately on confirmed payment.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Experience narrative: "they pay it by card on the spot" — instant, in-portal payment is central to the "get paid faster" value proposition. MVP phase: the product's payment promise depends on it.

**Connected Entities:** Payment (create), Invoice (update: paid state)

**Key Capabilities:**
- Pay by card or bank transfer — client completes payment from the invoice view
- Immediate status update — the invoice reflects Paid the moment payment is confirmed
- Failure handling — a failed or declined payment can be retried immediately

**Primary Flows & Alternates:**
- Happy path: Owen opens the invoice, pays by card or bank transfer, the invoice updates to Paid in both portal and dashboard
- Declined payment: Owen sees the failure reason and can retry immediately, no separate support step required

**States:** Empty: N/A — a payment view only exists for a sent invoice. Loading: payment processing shows a clear in-progress state rather than allowing a second submission. Error: a declined or failed payment shows the reason and a retry path; the invoice remains correctly Unpaid. Offline-degraded: payment requires connectivity; the client sees a clear "reconnect to pay" message.

**Validation & Limits:** A paid invoice cannot be paid again; partial payments are not supported in v1 — an invoice is either unpaid or paid in full.

**Access:** Owen (Own-only) can pay invoices for his own company; Priya has no access; Nadia sees payment status but never handles card details — the platform never touches card numbers or holds funds (BRIEF.md, Constraints).

**Communications:** Payment confirmation email to Owen and to Nadia when a payment succeeds.

**Data Notes:** Captured: payment timestamp, method, amount. Displayed: paid/unpaid/overdue status. Derived: none. Source: the payment-processing capability (BRIEF.md, Ecosystem & Integrations).

**Interactions:** Depends on Invoice Generation & Sending (FEAT-09); feeds Financial Dashboard (FEAT-12) and stops Automated Payment Reminders (FEAT-11) once paid.

**Signals:** payment_initiated, payment_succeeded, payment_failed, payment_retried.

### Automated Payment Reminders

**ID:** FEAT-11

**Description:** An overdue invoice automatically triggers polite reminder emails on day 3 and day 10 overdue, with no manual action from the freelancer, and stops the instant the invoice is paid.

**Priority:** Core

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** BRIEF.md, Experience narrative: "When an invoice goes overdue, polite reminders go out on day 3 and day 10 without you typing a word" — this is the brief's explicit answer to "the freelancer stops chasing" (BRIEF.md, Vision). MVP phase: central to the "get paid faster, chase less" promise.

**Connected Entities:** Invoice (read), Reminder Log (create)

**Key Capabilities:**
- Automatic reminders — day-3 and day-10 overdue emails with no manual trigger
- Pause per invoice — freelancer can pause reminders for a specific invoice (e.g., a dispute in progress)
- Automatic stop — reminders stop the moment the invoice is paid

**Primary Flows & Alternates:**
- Happy path: due date passes unpaid -> day-3 reminder sends -> if still unpaid, day-10 reminder sends -> reminders stop on payment
- Paused invoice: Nadia pauses reminders for one invoice without affecting any other invoice's schedule

**States:** Empty: N/A — reminders only exist against overdue invoices. Loading: N/A — a scheduled background behavior with no interactive loading state. Error: a failed reminder send is retried automatically and logged rather than silently dropped. Offline-degraded: N/A — this is a server-scheduled behavior independent of either party's connectivity at trigger time.

**Validation & Limits:** Exactly two scheduled reminders per overdue invoice (day 3, day 10) unless paused; reminders stop immediately on payment or freelancer pause.

**Access:** Nadia can view and pause/resume reminders for her invoices (Full). Owen receives reminders addressed to him with no configuration access. Priya is not addressed by billing reminders, per the Access Matrix.

**Communications:** The feature's entire purpose is a communication: polite overdue-payment emails to the Primary Contact at day 3 and day 10.

**Data Notes:** Captured: reminder-sent timestamps, pause state. Displayed: reminder history on the invoice detail. Derived: "overdue" status from due date vs. current date. Source: Invoice Generation & Sending (FEAT-09) due dates.

**Interactions:** Depends on Invoice Generation & Sending (FEAT-09) and Invoice Payment Processing (FEAT-10, to know when to stop); feeds Immutable Activity & Audit Trail (FEAT-13).

**Signals:** reminder_sent (day 3 / day 10), reminder_paused, reminder_resumed.

### Freelancer Financial Dashboard

**ID:** FEAT-12

**Description:** The freelancer sees earned, outstanding, and overdue totals across all clients and projects at a glance, and can drill into a single client or project's financial detail.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Experience narrative: "At month end you open your dashboard and see earned, outstanding and overdue, per client" — this is the brief's explicit description of the freelancer's core recurring need. MVP phase: the "money in one place" promise depends on it.

**Connected Entities:** Invoice (read), Payment (read), Project (read)

**Key Capabilities:**
- View aggregate totals — earned, outstanding, overdue across all clients
- Drill into a client or project — see the financial detail behind the aggregate
- See status at a glance — visually distinguish paid, due, and overdue

**Primary Flows & Alternates:**
- Happy path: open the dashboard -> see aggregate totals -> drill into a specific client/project for detail
- New account: a freelancer with no invoices yet sees a clear zero-state rather than a broken chart

**States:** Empty: no invoices yet shows an explanatory zero-state with a prompt toward sending a first proposal. Loading: totals render with a lightweight progress indicator while aggregating across many clients. Error: if aggregation fails, the last successfully computed totals are shown with a retry option, never a blank or misleading number. Offline-degraded: the most recently loaded totals remain viewable read-only.

**Validation & Limits:** No user input is captured here; the view spans the freelancer's full client roster (BRIEF.md, Scale) without a hard limit on history depth.

**Access:** Nadia only (Full, her own account). No client contact ever sees another client's totals or the freelancer's aggregate view. Dana (Support Operator) may view it read-only for support purposes, per the Access Matrix.

**Communications:** N/A — this is a self-initiated view; it sends no notifications itself.

**Data Notes:** Displayed: earned/outstanding/overdue totals per client and in aggregate. Derived: all totals are computed from Invoice and Payment records; nothing is captured directly in this view. Source: Invoice Generation & Sending (FEAT-09) and Invoice Payment Processing (FEAT-10).

**Interactions:** Depends on Invoice Generation & Sending (FEAT-09), Invoice Payment Processing (FEAT-10), and Client & Project Management (FEAT-01).

**Signals:** dashboard_viewed, client_drilldown_viewed, dashboard_empty_state_shown.

### Immutable Activity & Audit Trail

**ID:** FEAT-13

**Description:** Every proposal acceptance, milestone approval, and sent invoice is recorded permanently with who, what, and when, giving the freelancer a defensible record to point to if a scope dispute happens.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md, Success Criteria: "When a scope argument happens, the freelancer can point at the approval record" and Constraints: "accepted proposals, approvals and sent invoices must never be silently edited afterwards." This is the product's evidentiary backbone. MVP phase: without it, none of the record-immutability promises are verifiable.

**Connected Entities:** Activity Log Entry (create)

**Key Capabilities:**
- Automatic recording — every record-worthy event is logged without manual action
- Chronological project history — the full sequence of what happened, and when, per project
- Permanent record — no entry can be edited or deleted once written

**Primary Flows & Alternates:**
- Happy path: a record-worthy event occurs (acceptance, approval, invoice sent) -> an entry is written automatically -> Nadia can view the full history for any project
- Dispute: a client disputes an approval; Nadia opens the trail and points to the exact timestamped record

**States:** Empty: a brand-new project shows an empty trail with an explanation that entries will appear as milestones progress. Loading: the trail loads chronologically with a lightweight indicator for projects with long histories. Error: if the trail temporarily fails to load, previously fetched entries remain visible; nothing is ever removed by a failed request. Offline-degraded: the most recently loaded trail remains viewable read-only.

**Validation & Limits:** Entries are append-only — no entry can be edited or deleted once written; the trail has no depth limit within a project's lifetime.

**Access:** Nadia (Full, her own projects); Dana (Support Operator, View), per the Access Matrix. No client contact sees the full cross-event trail, though they see the outcome of their own actions within their scoped view.

**Communications:** N/A — the trail itself sends no notifications; it passively records events that trigger their own communications elsewhere.

**Data Notes:** Captured: event type, actor, timestamp, and a reference to the affected record. Displayed: a chronological project timeline. Derived: none — this is the authoritative record. Source: Proposal Acceptance (FEAT-03), Milestone Approval (FEAT-08), Invoice Generation & Sending (FEAT-09).

**Interactions:** Written to by FEAT-03, FEAT-08, FEAT-09, FEAT-11; read by Nadia directly and by In-App Notification Center (FEAT-29).

**Signals:** activity_entry_written, activity_trail_viewed.

### Notifications (Email)

**ID:** FEAT-14

**Description:** Every event that needs to reach a person — a new proposal, a deliverable ready for review, an approval request, an invoice, a reminder, a payment confirmation — is delivered by email, since clients will not install an app.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md, Ecosystem & Integrations: "all notifications to clients and freelancers go by email. Clients will not install an app." Every other feature's Communications field depends on this delivery mechanism existing. MVP phase: nothing client-facing can reach the client without it.

**Connected Entities:** Notification (create, send)

**Key Capabilities:**
- Event-triggered email dispatch — every notable event sends the correct email to the correct recipient
- Delivery tracking — failed or bounced sends are surfaced rather than silently lost

**Primary Flows & Alternates:**
- Happy path: a triggering event fires elsewhere in the product -> an email is composed and sent to the correct recipient(s) -> delivery status is tracked
- Delivery failure: an email fails to deliver (bad address, bounce); the freelancer sees a delivery warning on the affected project rather than the message silently vanishing

**States:** Empty: N/A — this is a background dispatch feature with no primary browsing view of its own. Loading: N/A. Error: a failed send is retried automatically a limited number of times, then surfaced to Nadia as a delivery warning. Offline-degraded: N/A — sending is server-side and independent of either party's connectivity.

**Validation & Limits:** Each notification type maps to exactly one triggering event, avoiding duplicate or missing emails; recipients are limited to contacts entitled to that event per the Access Matrix.

**Access:** Notification content and recipients follow the Access Matrix in user-persona.md — a Reviewer contact never receives an invoice email. Nadia manages her own notification preferences via Settings (FEAT-21).

**Communications:** This feature is the delivery mechanism for every other feature's Communications field; its own message is limited to delivery-failure warnings to Nadia.

**Data Notes:** Captured: notification type, recipient, send timestamp, delivery status. Displayed: delivery warnings surfaced to Nadia when relevant. Derived: none. Source: the triggering feature's event.

**Interactions:** Depended on by FEAT-02, FEAT-03, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11.

**Signals:** notification_sent, notification_delivery_failed, notification_bounced.

### Currency & Tax Handling

**ID:** FEAT-15

**Description:** The freelancer sets the currency for each client/project and shows an appropriate tax line on invoices, without any single country's currency or tax rules hard-coded into the product.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Scale & Non-Functional Expectations: "Currencies, tax on invoices (VAT, GST, US sales tax) and time zones must not be hard-coded" for a worldwide-from-day-one product. MVP phase: every invoice needs a correct currency and tax line from the first one issued.

**Connected Entities:** Invoice (read/update — currency and tax fields), Project (read — client's stated currency)

**Key Capabilities:**
- Set project currency — choose the billing currency per client/project
- Configure a tax line — set a tax label or rate (VAT, GST, US sales tax, or none) per client/region

**Primary Flows & Alternates:**
- Happy path: freelancer sets a project's currency and tax treatment during billing setup -> every invoice for that project uses it consistently
- Multiple countries: a freelancer serving clients in different countries sets a different currency/tax treatment per client without affecting others

**States:** Empty: a project with no currency set prompts the freelancer before the first invoice can be generated. Loading: N/A — configuration is local. Error: an invalid tax or currency configuration blocks invoice generation with a specific, correctable message rather than issuing an incorrect invoice. Offline-degraded: N/A — configuration is not a real-time or connectivity-sensitive action beyond normal saving.

**Validation & Limits:** Currency must be a recognized world currency; the exact depth of tax calculation (a shown line vs. calculated compliance) is left to the freelancer's configuration, since it is an open question in BRIEF.md's Open Questions.

**Access:** Nadia (Full) configures currency/tax per project; Owen (Own-only, view) sees the resulting amounts and tax line on his invoices; Priya has no invoice access.

**Communications:** N/A — configuration itself sends no notification; the resulting invoice carries its own (FEAT-09).

**Data Notes:** Captured: currency selection, tax label/rate per project. Displayed: on every invoice and on the dashboard's totals. Derived: none — displayed as configured. Source: freelancer configuration.

**Interactions:** Depended on by Invoice Generation & Sending (FEAT-09); read by Financial Dashboard (FEAT-12) for correct aggregation across currencies.

**Signals:** currency_set, tax_treatment_set, invoice_currency_mismatch_flagged.

### Large File Handling & Storage

**ID:** FEAT-16

**Description:** The product reliably accepts and delivers large files — tens of MB, sometimes over 1 GB for video — with resumable uploads, staying within the freelancer's stated infrastructure budget.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md, Scale & Non-Functional Expectations: "large files are the norm... typically tens of MB and sometimes over 1 GB for video," alongside a roughly $100/month infrastructure Constraint — this feature is the product's answer to that tension. MVP phase: Deliverable Upload & Sharing (FEAT-06) cannot honestly work without it.

**Connected Entities:** Deliverable (read/update — file storage), Deliverable Version (create)

**Key Capabilities:**
- Resumable large-file upload — uploads survive an interrupted connection
- Cost-conscious storage — file handling stays within the stated infrastructure budget
- Reliable delivery — clients stream or download without installing anything

**Primary Flows & Alternates:**
- Happy path: a deliverable upload begins -> large files upload in the background with visible progress -> the client streams or downloads directly
- Interrupted upload: a dropped connection pauses the upload; it resumes from where it left off rather than restarting

**States:** Empty: N/A — this is infrastructure behind Deliverable Upload & Sharing (FEAT-06), not a standalone screen. Loading: real progress and an estimated completion for large files, never an indefinite spinner. Error: a failed upload or download is retried automatically before surfacing a manual retry option. Offline-degraded: an interrupted upload pauses and resumes automatically on reconnection.

**Validation & Limits:** A per-file size ceiling and a per-freelancer storage allowance are enforced to keep infrastructure cost within budget until plan or usage changes that math; the freelancer is warned before hitting a limit.

**Access:** Follows Deliverable Upload & Sharing's Access field — Nadia (Full), Owen/Priya (Own-only, view/download).

**Communications:** N/A — this is infrastructure supporting FEAT-06's notifications, not a separate message source.

**Data Notes:** Captured: file bytes, size, upload/download timestamps. Displayed: none directly — surfaced through Deliverable Upload & Sharing (FEAT-06). Derived: storage usage totals per freelancer, used for plan-limit warnings. Source: the uploaded file itself.

**Interactions:** Supports Deliverable Upload & Sharing (FEAT-06) and Deliverable Version History (FEAT-17).

**Signals:** large_upload_started, large_upload_resumed, storage_limit_warning_shown.

## Important Features

### Deliverable Version History

**ID:** FEAT-17

**Description:** Every re-upload to a deliverable keeps the prior version rather than overwriting it, so both freelancer and client can see and open any earlier round rather than losing track in a folder of "final_v3_REAL" files.

**Priority:** Important

**Phase:** v1

**Type:** User-Facing

**Rationale:** BRIEF.md's Problem Statement names this exact failure mode by name ("final_v3_REAL" versions). Ranked Important rather than Core because a single-round deliverable still works without it; phased to v1 since the MVP loop (upload -> comment -> approve) must be proven first, and version confusion only becomes visible once revisions start happening at scale.

**Connected Entities:** Deliverable Version (create, read)

**Key Capabilities:**
- Preserve every version — a re-upload never overwrites a prior round
- Browse by version — open and compare any round by number
- Version-anchored comments — feedback stays attached to the version it was made on

**Primary Flows & Alternates:**
- Happy path: freelancer uploads a revised file to an existing deliverable -> the prior version is preserved and labeled -> both parties open any version by round number
- Comment on an old version: a client comments on an older version after a newer one exists; the comment stays attached to the version it was made on

**States:** Empty: a deliverable with only one version shows no version selector, avoiding clutter until needed. Loading: switching between versions shows a brief loading indicator for large files. Error: a failed version upload does not overwrite the existing version — the prior version remains intact and current. Offline-degraded: previously loaded versions remain viewable read-only.

**Validation & Limits:** No fixed cap on version count per deliverable within the plan's storage allowance (FEAT-16); each version is immutable once uploaded.

**Access:** Follows Deliverable Upload & Sharing's Access field — Nadia (Full), Owen/Priya (Own-only, view).

**Communications:** Reuses Deliverable Upload & Sharing's "new deliverable ready" notification (FEAT-06) for each new version.

**Data Notes:** Captured: each version's file, upload timestamp, round number. Displayed: version selector on the deliverable view. Derived: "latest version" is the most recent upload. Source: freelancer uploads over time.

**Interactions:** Depends on Deliverable Upload & Sharing (FEAT-06) and Large File Handling & Storage (FEAT-16); referenced by Deliverable Review & Feedback (FEAT-07) for per-version comments.

**Signals:** version_uploaded, version_opened, version_compared.

### Client Contact Management & Roles

**ID:** FEAT-18

**Description:** The freelancer adds contacts to a client company and assigns each one the Primary or Reviewer role; a Primary contact can invite additional colleagues at their own company.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Target Users & Roles: "not every contact may approve work or see invoices" with the founder's leaning toward Primary (accept/approve/pay) versus Reviewer (comment-only) contacts. Ranked Important rather than Core because the underlying access differentiation is what makes every Core feature's Access field correct, but the management surface itself is a supporting capability. Phased MVP because the role distinction must exist from the first client onward for the Access Matrix to hold.

**Connected Entities:** Client Contact (create, update, remove)

**Key Capabilities:**
- Add a contact — freelancer records a new person at a client company
- Assign a role — Primary or Reviewer, per BRIEF.md's leaning
- Primary-led invites — a Primary contact can invite Reviewer colleagues at their own company

**Primary Flows & Alternates:**
- Happy path: freelancer adds a client's first contact as Primary -> that Primary contact can invite Reviewer colleagues from their own portal view
- Role change: a contact's role is changed after the fact (e.g., a second Primary is added for co-founders); the change applies to future actions only, never retroactively altering a past approval
- Removing the last Primary: blocked until a replacement Primary is designated, since a proposal cannot be accepted without one

**States:** Empty: a client with no contacts yet blocks sending a proposal until at least one Primary contact exists, with a clear prompt. Loading: N/A — local list. Error: a failed add/invite preserves entered contact details for retry. Offline-degraded: N/A — contact management is an infrequent, connectivity-required action.

**Validation & Limits:** A client company must have at least one Primary contact before a proposal can be sent; a contact's email must be unique within that client company.

**Access:** Nadia (Full) manages contacts and roles for any client. Owen (Own-only) can invite Reviewer contacts at his own company, per the Access Matrix. Priya has no contact-management access.

**Communications:** Invitation email to a newly added contact with their first magic-link sign-in.

**Data Notes:** Captured: contact name, email, role, inviting party. Displayed: contact list per client with role labels. Derived: none. Source: freelancer or Primary-contact input.

**Interactions:** Depends on Client & Project Management (FEAT-01); gates Client Portal Access (FEAT-05) recognition and every role-differentiated Access field above.

**Signals:** contact_added, contact_role_changed, contact_invited_by_primary, contact_removed.

### Freelancer Branding

**ID:** FEAT-19

**Description:** The freelancer sets a logo and brand colour so every client-facing screen looks like her own practice rather than a generic shared tool.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Ecosystem & Integrations: "each freelancer's own logo and colours on their portal." Ranked Important rather than Core because the product functions correctly, if generically, without it; phased MVP because BRIEF.md's growth loop depends on clients seeing a branded, professional portal from the very first project ("new freelancers arrive after a client or peer sees someone else's portal," BRIEF.md, Business Context).

**Connected Entities:** Branding Profile (create, update)

**Key Capabilities:**
- Set a logo — upload the freelancer's own logo
- Set a brand colour — choose a primary colour applied across client-facing screens

**Primary Flows & Alternates:**
- Happy path: freelancer uploads a logo and picks a brand colour in settings -> every client-facing screen reflects it immediately
- No branding set: the portal falls back to a clean, neutral default rather than looking broken or generically unfinished

**States:** Empty: no logo/colour set yet shows the clean neutral default (BRIEF.md, Constraints: "clean and professional with lots of white space"). Loading: N/A — a small configuration form. Error: a failed logo upload preserves the previous branding until a new one succeeds. Offline-degraded: N/A — branding is set from the freelancer's desktop with expected connectivity.

**Validation & Limits:** Logo file size and format are limited to keep client-facing pages fast to load on mobile; one logo and one primary colour per freelancer account in v1.

**Access:** Nadia (Full); client contacts see the resulting branding but cannot change it.

**Communications:** N/A — branding itself sends no notification; it affects the appearance of every client-facing communication and screen.

**Data Notes:** Captured: logo file, brand colour value. Displayed: on every client-facing surface. Derived: none. Source: freelancer upload/selection.

**Interactions:** Affects the presentation of Proposal Creation & Sending (FEAT-02), Deliverable Upload & Sharing (FEAT-06), Invoice Generation & Sending (FEAT-09), and Client Portal Access (FEAT-05).

**Signals:** branding_logo_uploaded, branding_color_set, branding_reset_to_default.

### Onboarding / First-Run Setup

**ID:** FEAT-20

**Description:** A new freelancer is guided from signing up to sending her first proposal in one sitting: add a first client, set basic branding, and draft the first proposal.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** The decomposition checklist's Commonly Forgotten Areas require a decided first-run experience; the brief's three-month, first-paying-freelancer constraint (BRIEF.md, Constraints) makes a smooth first session essential rather than optional. Ranked Important rather than Core because the product's ongoing value does not depend on onboarding once the first project exists; phased MVP since a confusing first session directly threatens the founder's own three-month goal.

**Connected Entities:** N/A — onboarding orchestrates existing entities rather than owning one of its own.

**Key Capabilities:**
- Guided first client and project — walks the freelancer through her first setup
- Optional branding step — offered but skippable
- Guided first proposal — leads into drafting the first real proposal

**Primary Flows & Alternates:**
- Happy path: freelancer signs up -> is guided through adding a first client, setting basic branding, and starting a first proposal -> lands in the normal dashboard once the first project exists
- Skipped step: freelancer skips branding and returns to it later from Settings (FEAT-21) without penalty

**States:** Empty: this is itself the empty-state experience for a brand-new account. Loading: N/A — a short guided sequence. Error: a failed step (e.g., logo upload) does not block progressing to the next step; it can be completed later. Offline-degraded: onboarding requires connectivity to create the first client/project records.

**Validation & Limits:** Only adding a first client and project is mandatory to exit onboarding; branding and full billing setup are optional at this stage.

**Access:** Nadia only — onboarding is a freelancer-side experience; client contacts have no equivalent beyond their first magic-link login (FEAT-05).

**Communications:** A welcome email confirming account creation.

**Data Notes:** Captured: whichever fields the freelancer completes during the guided steps. Displayed: progress through the guided sequence. Derived: none. Source: freelancer input during setup.

**Interactions:** Orchestrates Client & Project Management (FEAT-01), Freelancer Branding (FEAT-19), and Proposal Creation & Sending (FEAT-02).

**Signals:** onboarding_started, onboarding_step_completed, onboarding_completed, onboarding_step_skipped.

### Settings & Account Management

**ID:** FEAT-21

**Description:** The freelancer edits her account profile, manages notification preferences, and manages her own login recovery (clients use magic links only, per FEAT-05).

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** The decomposition checklist's Commonly Forgotten Areas expect a settings surface for any product with accounts. Ranked Important rather than Core because the product's primary loop functions without visiting settings; phased MVP since account-recovery and notification control are needed from first use.

**Connected Entities:** Freelancer Account (update)

**Key Capabilities:**
- Edit profile — update account details
- Manage notification preferences — control which optional notifications are sent
- Manage login recovery — update email/login method for her own account

**Primary Flows & Alternates:**
- Happy path: freelancer opens settings -> edits profile, notification preferences, or account details -> changes save immediately
- Account closure: a freelancer wanting to close her account entirely is routed to Data Export & Account Deletion (FEAT-24) rather than duplicating that flow here

**States:** Empty: N/A — settings always has default values from account creation. Loading: N/A — a standard form. Error: a failed save is retried without discarding unsaved fields. Offline-degraded: N/A — settings changes require connectivity to persist.

**Validation & Limits:** Email changes require re-verification; notification preferences cannot disable transactional emails core to the record (e.g., a payment confirmation always sends).

**Access:** Nadia (Full, her own account only); client contacts have no equivalent settings surface.

**Communications:** Confirmation email on account-critical changes (email address change, login method change).

**Data Notes:** Captured: profile fields, notification preferences. Displayed: current settings values. Derived: none. Source: freelancer input.

**Interactions:** Affects delivery behavior of Notifications (FEAT-14); independent of client-facing features.

**Signals:** settings_updated, notification_preference_changed, account_email_changed.

### Accounting Export

**ID:** FEAT-22

**Description:** The freelancer exports invoices and payments as a CSV or a QuickBooks/Xero-compatible file for her own bookkeeping.

**Priority:** Important

**Phase:** v1

**Type:** User-Facing

**Rationale:** BRIEF.md, Ecosystem & Integrations: "v1 provides an export file (CSV or a QuickBooks/Xero-compatible format). No live sync." Ranked Important because bookkeeping is a real recurring need but not part of the core client-facing loop; phased v1, matching the brief's own stated v1 scope for this integration.

**Connected Entities:** Accounting Export File (create)

**Key Capabilities:**
- Select a date range — scope the export to a period
- Generate a file — CSV or QuickBooks/Xero-compatible format
- Download — pull the file into her own accounting software

**Primary Flows & Alternates:**
- Happy path: freelancer selects a date range -> generates an export file -> downloads it to import into her own accounting software
- No data in range: an empty date range produces a clear "nothing to export" result rather than a broken empty file

**States:** Empty: no invoices in the selected range shows a plain explanatory message. Loading: generation shows progress for large date ranges. Error: a failed generation is retried without corrupting a partial file. Offline-degraded: N/A — export requires connectivity to generate and download.

**Validation & Limits:** Export covers only the freelancer's own data; date range is bounded by the account's actual invoice history.

**Access:** Nadia only (Full, her own data); no client contact has export access.

**Communications:** N/A — this is a self-initiated download; no notification is sent.

**Data Notes:** Captured: none — this reads existing records. Displayed: a downloadable file. Derived: the export format is derived from stored invoice and payment data. Source: Invoice Generation & Sending (FEAT-09) and Invoice Payment Processing (FEAT-10).

**Interactions:** Depends on FEAT-09 and FEAT-10 for the data it exports.

**Signals:** export_generated, export_downloaded, export_empty_result_shown.

### Subscription Plan & Billing Management

**ID:** FEAT-23

**Description:** The freelancer sees her current Clientroom plan (free for one or two clients, a flat monthly or yearly price above that) and upgrades when she adds more clients.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** BRIEF.md, Business Context: "sold as a subscription per freelancer, priced by number of active clients: free for one or two clients, then one flat monthly or yearly price." Ranked Important rather than Core because it governs the freelancer's own relationship with the product, not the client-facing value loop; phased MVP because the founder's three-month first-paying-freelancer goal (BRIEF.md, Constraints) requires billing to exist at launch.

**Connected Entities:** Subscription Plan (create, update)

**Key Capabilities:**
- View current plan — see tier and usage against its client limit
- Upgrade — subscribe to the paid tier when exceeding the free-tier client count
- Downgrade offer — reduce plan when client count drops back below the threshold

**Primary Flows & Alternates:**
- Happy path: freelancer adds a client beyond the free-tier limit -> is prompted to upgrade -> subscribes at the flat monthly or yearly price -> gains the corresponding client capacity
- Reduced usage: a freelancer on a paid plan drops below the free-tier threshold; she is offered, not forced, a downgrade

**States:** Empty: a brand-new account starts on the free tier automatically, no explicit "no plan" state. Loading: N/A — plan status is a simple read. Error: a failed subscription charge shows a clear reason and retry, and does not silently lock the freelancer out of already-active client work mid-session. Offline-degraded: N/A — billing changes require connectivity.

**Validation & Limits:** The free tier is capped at one or two active clients (exact number confirmed at pricing finalization, per BRIEF.md's Open Questions); exceeding the cap requires an active paid subscription before a new client beyond the limit can be added.

**Access:** Nadia only (Full, her own subscription); this has no client-facing surface.

**Communications:** Email confirmation on plan change, upgrade, or a failed subscription charge.

**Data Notes:** Captured: plan tier, active-client count, billing cycle. Displayed: current plan and usage against its client limit. Derived: whether the account is within its plan's client limit. Source: freelancer's own client roster (FEAT-01) and the subscription-billing capability.

**Interactions:** Reads Client & Project Management (FEAT-01) for active-client count; independent of client Invoicing & Payments (FEAT-09/FEAT-10), which is the freelancer's own client's money, never the platform's.

**Signals:** plan_viewed, plan_upgraded, plan_downgrade_offered, subscription_charge_failed.

### Data Export & Account Deletion

**ID:** FEAT-24

**Description:** The freelancer can export all of her own data and permanently delete her account and its data.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** BRIEF.md, Constraints: "freelancers must be able to export and delete their data. Personal data of worldwide clients falls under GDPR." Ranked Important because it is a compliance and trust requirement rather than part of the daily value loop; phased MVP since this is a hard regulatory requirement that must exist from launch, not added later.

**Connected Entities:** N/A — this feature acts across all of a freelancer's entities rather than owning a new one.

**Key Capabilities:**
- Full data export — a complete archive of clients, projects, proposals, invoices, and activity trail
- Account deletion — permanently remove the account and its data, with explicit confirmation

**Primary Flows & Alternates:**
- Happy path: freelancer requests a full data export -> receives a complete archive -> separately requests account deletion, confirmed explicitly, which permanently removes her data
- Open items at deletion time: a freelancer with active unpaid invoices or pending approvals is warned about the consequences before deletion is finalized, not silently blocked

**States:** Empty: N/A — this always operates on whatever data exists, even a near-empty account. Loading: export generation and deletion processing both show progress for accounts with substantial history. Error: a failed export is retried without partial, corrupted output; a failed deletion leaves the account fully intact rather than half-deleted. Offline-degraded: N/A — both actions require connectivity.

**Validation & Limits:** Account deletion requires explicit confirmation given its irreversibility; deletion honors any legal retention requirement for financial records before final purge.

**Access:** Nadia only (Full, her own account); no client contact can export or delete the freelancer's account. A client contact's own personal data (email, comments) is included in the freelancer's export and removed on account deletion, per GDPR-class handling (BRIEF.md, Privacy).

**Communications:** Confirmation email when an export is ready to download; a final confirmation email before deletion is irreversibly completed.

**Data Notes:** Captured: none — this reads and then removes existing records. Displayed: export download link; deletion confirmation state. Derived: none. Source: the full breadth of the freelancer's existing data across all features.

**Interactions:** Reads from every feature that stores freelancer or client data; on deletion, removes records referenced across FEAT-01 through FEAT-23.

**Signals:** data_export_requested, data_export_ready, account_deletion_requested, account_deletion_completed.

### Refund & Cancelled Project Handling

**ID:** FEAT-25

**Description:** The freelancer marks an invoice as refunded (the refund itself is issued through her own processor account) and can mark a project cancelled, preserving the record rather than deleting it.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Open Questions notes that "the refund, chargeback and cancelled-project flow was not defined," while Constraints requires "payments and records must be correct" and that records are never silently altered. Ranked Important rather than Core because most projects never need it; phased MVP because payments begin at MVP (FEAT-10), so a way to record a refund or cancellation correctly must exist from the same point, even in minimal form, to keep the record trustworthy.

**Connected Entities:** Invoice (update: refunded/cancelled state), Project (update: cancelled state)

**Key Capabilities:**
- Mark an invoice refunded — records that a refund was issued outside the platform, through the freelancer's own processor
- Mark a project cancelled — records that work has stopped, without deleting history

**Primary Flows & Alternates:**
- Happy path: freelancer issues a refund directly through her processor -> marks the corresponding invoice as Refunded in Clientroom so records stay accurate -> optionally marks the project Cancelled if work has stopped
- Manual dispute resolution: a dispute resolved outside the platform (the full flow is an open question per BRIEF.md) is recorded as a note against the invoice/project so the trail stays truthful even where the underlying workflow is manual

**States:** Empty: N/A — this only applies to an existing invoice/project. Loading: N/A — a status update. Error: a failed status update is retried without leaving the invoice in an ambiguous state. Offline-degraded: N/A — status changes require connectivity to persist reliably as part of the record.

**Validation & Limits:** A refunded invoice cannot later be marked Paid without a clear, logged correction; a cancelled project retains its full history rather than being deleted.

**Access:** Nadia only (Full); Owen sees the resulting Refunded/Cancelled status on his own invoices/projects (Own-only, view); Priya has no billing visibility.

**Communications:** Notification email to Owen when an invoice is marked refunded or a project is marked cancelled.

**Data Notes:** Captured: refund/cancellation timestamp, freelancer-entered reason (optional). Displayed: Refunded/Cancelled status alongside the original invoice/project record, never overwriting it. Derived: none. Source: freelancer input, reflecting an action taken directly through the payment processor.

**Interactions:** Depends on Invoice Generation & Sending (FEAT-09) and Invoice Payment Processing (FEAT-10); feeds Immutable Activity & Audit Trail (FEAT-13) and Financial Dashboard (FEAT-12) totals.

**Signals:** invoice_marked_refunded, project_marked_cancelled.

## Nice-to-Have Features

### Legally Binding E-Signature for Proposals

**ID:** FEAT-26

**Description:** Upgrades the recorded "Accept" click to a legally binding e-signature for freelancers who want stronger contractual weight than a timestamp alone.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** BRIEF.md's Open Questions leaves unresolved "do proposals need legally binding e-signatures, or is a recorded, timestamped 'Accept' enough?" — the founder's own framing treats the timestamped Accept as the presumptive default. Nice-to-Have because the MVP default (FEAT-03) already satisfies the brief's evidence requirement; phased Later since it can be added per-proposal without restructuring the acceptance flow.

**Connected Entities:** Proposal (update: signature record)

**Key Capabilities:**
- Opt-in e-signature — freelancer enables it for a specific proposal
- Signing step — client completes a signature step instead of a plain Accept click

**Primary Flows & Alternates:**
- Happy path: freelancer enables e-signature for a proposal -> client completes a signing step -> the signed record is stored alongside the timestamp
- Not enabled: the proposal falls back to the standard Accept flow (FEAT-03), the v1 default per BRIEF.md's Open Questions

**States:** Empty: N/A — appears only when a freelancer opts in per proposal. Loading: N/A — a short signing step. Error: a failed signature submission preserves entered signature data for retry. Offline-degraded: signing requires connectivity, consistent with its legal-record purpose.

**Validation & Limits:** Available per proposal, not forced account-wide; once signed, the record is immutable like any other acceptance.

**Access:** Owen (Primary Contact) can sign, same access boundary as standard acceptance (FEAT-03); Priya cannot.

**Communications:** A signed-copy confirmation email to both parties, in addition to the standard acceptance confirmation (FEAT-03).

**Data Notes:** Captured: signature data, signer identity, timestamp. Displayed: a "Signed on {date}" marker, distinct from a plain "Accepted." Derived: none. Source: the signing action.

**Interactions:** Extends Proposal Acceptance (FEAT-03); feeds Immutable Activity & Audit Trail (FEAT-13).

**Signals:** esignature_enabled, proposal_signed.

### Custom Domain per Freelancer

**ID:** FEAT-27

**Description:** A freelancer points her own domain at her portal so clients see the freelancer's brand end-to-end.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** Platform

**Rationale:** BRIEF.md, Ecosystem & Integrations: "ideally their own custom domain (timing is an open question)." Nice-to-Have because the shared portal already fulfills the branding promise (FEAT-19) without it; phased Later as the brief itself defers the timing question.

**Connected Entities:** Custom Domain Record (create, update)

**Key Capabilities:**
- Add a domain — freelancer enters her own domain
- Verify and go live — once verified, the portal is reachable at the freelancer's own domain

**Primary Flows & Alternates:**
- Happy path: freelancer enters her domain -> follows verification steps -> once verified, her portal is reachable at her own domain
- Verification failure: a misconfigured record is named specifically to the freelancer, and the shared default domain keeps working in the meantime

**States:** Empty: no custom domain configured — the freelancer uses the shared default domain with no disruption. Loading: verification shows a pending state while domain records propagate. Error: a failed verification names the specific problem and offers re-check. Offline-degraded: N/A — this is a one-time configuration step, not an ongoing runtime dependency for either party's session.

**Validation & Limits:** One custom domain per freelancer account; the shared default domain always remains available as a fallback.

**Access:** Nadia only (Full); client contacts simply experience whichever domain is active.

**Communications:** Confirmation email once the domain is verified and live.

**Data Notes:** Captured: the domain name and its verification state. Displayed: verification status and instructions. Derived: none. Source: freelancer input plus domain-verification confirmation.

**Interactions:** Affects how Client Portal Access (FEAT-05) and all client-facing links are presented; typically adopted together with Freelancer Branding (FEAT-19).

**Signals:** custom_domain_added, custom_domain_verified, custom_domain_verification_failed.

### Global Search Across Clients & Projects

**ID:** FEAT-28

**Description:** The freelancer searches across all clients, projects, proposals, and invoices from one search box instead of navigating the client list manually.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** User-Facing

**Rationale:** The decomposition checklist's Cross-Cutting Concerns flags search once a product manages more than one entity type, which this product clearly does. Nice-to-Have because a freelancer with 3–15 clients (BRIEF.md, Scale) can still browse manually without it; phased v1 since it becomes genuinely useful once a freelancer has accumulated enough clients and history to need it.

**Connected Entities:** N/A — search reads across existing entities rather than owning one.

**Key Capabilities:**
- Search across entity types — clients, projects, proposals, invoices in one query
- Jump to a result — selecting a result opens it directly

**Primary Flows & Alternates:**
- Happy path: freelancer types a query -> matching clients, projects, proposals, and invoices appear ranked by relevance -> selecting a result opens it directly
- No matches: a clear "no results" state rather than an empty blank area

**States:** Empty: the search box itself has no empty state beyond an unused input; results panel shows "no results" when a query matches nothing. Loading: results show a lightweight in-progress indicator as the freelancer types. Error: a failed search retries automatically before surfacing a manual retry. Offline-degraded: search falls back to whatever was most recently loaded locally, clearly marked as possibly stale.

**Validation & Limits:** Search scope is limited to the freelancer's own data (never another freelancer's, per BRIEF.md's Privacy constraint); minimum 2-character query to avoid overly broad result sets.

**Access:** Nadia only (Full, her own data); no client contact has this cross-account search.

**Communications:** N/A — search sends no notifications.

**Data Notes:** Captured: none — search reads existing records. Displayed: ranked results across entity types. Derived: relevance ranking. Source: Client & Project Management (FEAT-01), Proposal Creation & Sending (FEAT-02), Invoice Generation & Sending (FEAT-09).

**Interactions:** Reads across FEAT-01, FEAT-02, FEAT-06, FEAT-09.

**Signals:** search_performed, search_result_selected, search_no_results_shown.

### In-App Notification Center

**ID:** FEAT-29

**Description:** The freelancer sees a running feed of recent activity (approvals, payments, comments) inside the product, complementing the email notifications from FEAT-14.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** Platform

**Rationale:** The decomposition checklist's Cross-Cutting Concerns flags in-app notifications as a candidate alongside email; BRIEF.md establishes email as the required channel for clients, leaving an in-app feed as a freelancer-side convenience only. Nice-to-Have and phased Later since email already delivers every message the brief requires.

**Connected Entities:** Notification (read — surfaces existing records)

**Key Capabilities:**
- Recent-activity feed — a chronological view of recent events across all clients
- Mark read/open — triage the feed and jump to the related project

**Primary Flows & Alternates:**
- Happy path: freelancer opens the notification center -> sees recent events across all clients in one chronological feed -> marks items read or opens the related project directly
- No recent activity: a calm empty state rather than an implication that something is broken

**States:** Empty: no recent activity shows a plain "all caught up" message. Loading: the feed loads with a lightweight indicator for accounts with heavy recent activity. Error: a failed feed load falls back to the last successfully loaded feed. Offline-degraded: the last loaded feed remains viewable read-only.

**Validation & Limits:** Feed shows a rolling recent window, not unlimited history — the full record lives in the Activity & Audit Trail (FEAT-13); items can be marked read/unread but not deleted.

**Access:** Nadia only (Full, her own feed); client contacts have no equivalent in-app feed.

**Communications:** N/A — this surfaces notifications already sent by FEAT-14; it does not send new ones.

**Data Notes:** Displayed: a read/unread feed of recent events. Derived: entirely derived from events recorded by Immutable Activity & Audit Trail (FEAT-13) and Notifications (FEAT-14). Source: FEAT-13 and FEAT-14.

**Interactions:** Reads from Immutable Activity & Audit Trail (FEAT-13) and Notifications (FEAT-14).

**Signals:** notification_center_opened, notification_marked_read, notification_item_opened.

### Contextual Help & Guidance

**ID:** FEAT-30

**Description:** Light contextual tooltips and a short help reference for both the freelancer's dashboard and the client-facing portal, so first-time users of either side need no external documentation.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** Lifecycle

**Rationale:** The decomposition checklist's Commonly Forgotten Areas expects a decided answer on how users learn the product. Nice-to-Have because both sides' flows are designed to be self-explanatory in the moment (one-click accept, one-click approve); phased Later since it is a polish layer added once the core flows are proven rather than a launch requirement.

**Connected Entities:** N/A — a guidance layer over existing screens, not a data-owning entity.

**Key Capabilities:**
- On-demand explanation — a brief inline explanation for an unfamiliar control
- Permanent dismissal — an experienced user dismisses guidance and it does not reappear

**Primary Flows & Alternates:**
- Happy path: a first-time user encounters an unfamiliar control (e.g., a client seeing "Approve" for the first time) -> a brief inline explanation is available on demand -> the user proceeds without needing outside help
- Dismissal: an experienced user dismisses guidance permanently for that area

**States:** Empty: N/A — guidance appears only on demand or on first encounter, never as a standalone screen. Loading: N/A — static contextual content. Error: N/A — there is no failure mode for static help text. Offline-degraded: help content already loaded remains available; content not yet loaded is simply unavailable until reconnection.

**Validation & Limits:** Guidance is advisory only and never blocks a user from proceeding without reading it.

**Access:** Available to both Nadia and client contacts wherever relevant, since both sides are new-user audiences.

**Communications:** N/A — this is in-context content, not a message.

**Data Notes:** Captured: dismissal state per user, so dismissed tips do not reappear. Displayed: contextual tooltips/short explanations. Derived: none. Source: static product content, not user or client data.

**Interactions:** Overlays Onboarding / First-Run Setup (FEAT-20), Milestone Approval (FEAT-08), and Client Portal Access (FEAT-05) at their first-encounter moments.

**Signals:** help_tip_shown, help_tip_dismissed.

## Feature Interaction Summary

| Feature | Depends On |
|---------|------------|
| FEAT-01 Client & Project Management | None |
| FEAT-02 Proposal Creation & Sending | FEAT-01, FEAT-15 |
| FEAT-03 Proposal Acceptance | FEAT-02 |
| FEAT-04 Milestone & Payment Schedule Setup | FEAT-03 |
| FEAT-05 Client Portal Access (Magic-Link Login) | FEAT-18 |
| FEAT-06 Deliverable Upload & Sharing | FEAT-04, FEAT-16 |
| FEAT-07 Deliverable Review & Feedback | FEAT-06 |
| FEAT-08 Milestone Approval | FEAT-06, FEAT-07 |
| FEAT-09 Invoice Generation & Sending | FEAT-03, FEAT-08, FEAT-15 |
| FEAT-10 Invoice Payment Processing | FEAT-09 |
| FEAT-11 Automated Payment Reminders | FEAT-09, FEAT-10 |
| FEAT-12 Freelancer Financial Dashboard | FEAT-01, FEAT-09, FEAT-10 |
| FEAT-13 Immutable Activity & Audit Trail | FEAT-03, FEAT-08, FEAT-09, FEAT-11 |
| FEAT-14 Notifications (Email) | FEAT-02, FEAT-03, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11 |
| FEAT-15 Currency & Tax Handling | None |
| FEAT-16 Large File Handling & Storage | None |
| FEAT-17 Deliverable Version History | FEAT-06, FEAT-16 |
| FEAT-18 Client Contact Management & Roles | FEAT-01 |
| FEAT-19 Freelancer Branding | None |
| FEAT-20 Onboarding / First-Run Setup | FEAT-01, FEAT-02, FEAT-19 |
| FEAT-21 Settings & Account Management | FEAT-14 |
| FEAT-22 Accounting Export | FEAT-09, FEAT-10 |
| FEAT-23 Subscription Plan & Billing Management | FEAT-01 |
| FEAT-24 Data Export & Account Deletion | FEAT-01 through FEAT-23 |
| FEAT-25 Refund & Cancelled Project Handling | FEAT-09, FEAT-10 |
| FEAT-26 Legally Binding E-Signature for Proposals | FEAT-03 |
| FEAT-27 Custom Domain per Freelancer | FEAT-05, FEAT-19 |
| FEAT-28 Global Search Across Clients & Projects | FEAT-01, FEAT-02, FEAT-06, FEAT-09 |
| FEAT-29 In-App Notification Center | FEAT-13, FEAT-14 |
| FEAT-30 Contextual Help & Guidance | FEAT-08, FEAT-20, FEAT-05 |
