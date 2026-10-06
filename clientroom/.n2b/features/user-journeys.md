---
document_type: user-journeys
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed (2 fixes applied)
---

# User Journeys

## Journey

This document contains 11 journeys covering the full lifecycle across both sides of the product: the freelancer's setup, sales, delivery, billing, support, and account management, and the client's first login and feedback experience. Journeys are owned by Nadia (Freelancer), Owen (Client Primary Contact), and Priya (Client Reviewer Contact), with Dana (Support Operator) taking part in one. Every Core and Important feature from product-features.md appears in at least one journey, and all three coverage types (First-use, Regular, Edge/Recovery) are represented. [MODIFIED: one journey added (Getting Help Without Giving Up Control) and the first-time setup journey extended so the features added during synthesis — Payment Account Connection, Portal Referral Attribution, Operator Support Access — each appear in a journey]

---

### Freelancer First-Time Setup

**Owning Persona:** Nadia

**Coverage:** First-use

**Journey Goal:** Get from signing up to a first sent proposal in one sitting, with the portal already looking like her own practice.

**Entry Point:** Nadia signs up for Clientroom for the first time, motivated to move her next client off email and spreadsheets. She found it by following the small "Made with Clientroom" link on a fellow designer's client portal. [MODIFIED: entry point tied to the portal-driven growth loop in BRIEF.md's Business Context, now delivered by Portal Referral Attribution]

**Steps:**

1. First open — Nadia sees a short, guided setup rather than a blank dashboard, starting with an optional "How did you hear about us?" question that already knows which portal referred her. A welcome email confirms her account. [AUDIT-ADDED: 1 -- growth-loop attribution step]
2. Add first client and project — She is guided to add her first client company and create a project under it.
3. Set branding — She uploads her logo and picks a brand colour; the guided flow makes this feel optional, not mandatory.
4. Connect payments — She is offered a short step to connect her own payment-processor account, so her first client can pay the deposit by card on the spot; the status shows "Ready to accept payments." [AUDIT-ADDED: 1 -- value-flow walk: payment connection must happen before the first deposit can be paid]
5. Draft the first proposal — She is guided into drafting scope and price for the new project.
6. Ready state — Onboarding completes once the first client, project, and draft proposal exist; she lands in the normal dashboard, ready to send.

**Failure/Recovery Variant:** Nadia skips the branding step, unsure of her colours yet. The guided flow lets her continue to the proposal draft without penalty, and the portal falls back to a clean neutral default in the meantime — she finishes branding later from Settings with no lost progress.

**Success Outcome:** Within one sitting, Nadia has her first client, project, and a proposal ready to send, and understands what daily use will look like. [RESEARCH-INFORMED: competitors' setup takes 15–25+ hours (Dubsado, SuiteDash reviews, HIGH); this journey has nothing to configure beyond the essentials]

**Connected Features:** Onboarding / First-Run Setup, Client & Project Management, Freelancer Branding, Proposal Creation & Sending, Payment Account Connection, Portal Referral Attribution

---

### Send a Proposal and Get the Deposit Paid

**Owning Persona:** Nadia, Owen

**Coverage:** Regular

**Journey Goal:** Send a proposal for a new project and have the client accept it and pay the deposit, all within a day.

**Entry Point:** Nadia has a new client ready to start; she opens Clientroom to send them a proposal.

**Steps:**

1. Draft and send — Nadia sets the scope, price, and currency, then sends the proposal. Owen receives an email with the proposal link.
2. First client login — Owen requests a magic-link sign-in from the email and lands directly in his company's scoped view — no account to create.
3. Review and accept — Owen reads the scope and price and clicks Accept. The acceptance is timestamped immediately.
4. Deposit invoice appears — A deposit invoice is generated automatically and emailed to Owen with a pay link.
5. Pay on the spot — Owen pays by card directly from the portal; the money goes straight into Nadia's own connected payment account, and the invoice updates to Paid instantly for both Owen and Nadia. [RESEARCH-INFORMED: no platform fee or payout delay, in contrast to the stacked fees and held payouts reported for HoneyBook and Bonsai]

**Failure/Recovery Variant:** Owen's magic link expires before he opens it (he was traveling). He requests a fresh link in one step from the same email, with no loss of the proposal state — it is exactly as Nadia sent it.

**Success Outcome:** The client has accepted the proposal and paid the deposit within a day of receiving it, with a timestamped record on both sides and no email back-and-forth.

**Connected Features:** Proposal Creation & Sending, Client Portal Access (Magic-Link Login), Proposal Acceptance, Currency & Tax Handling, Invoice Generation & Sending, Invoice Payment Processing, Payment Account Connection, Client Contact Management & Roles, Notifications (Email)

---

### Deliver a Milestone Round and Collect Feedback

**Owning Persona:** Nadia, Priya

**Coverage:** Regular

**Journey Goal:** Upload a round of work to a milestone and get specific, contextual feedback from the client instead of scattered screenshots.

**Entry Point:** Nadia has finished the first round of design work for a milestone and is ready to share it.

**Steps:**

1. Set the milestone — Nadia had earlier defined this project's milestones and payment triggers; this round belongs to milestone 2.
2. Upload the deliverable — She uploads a large design file; upload progress is visible and the file completes reliably.
3. Client notified — Priya, the marketing lead, receives an email that a new deliverable is ready to review.
4. Review on mobile — Priya opens the portal on her phone and leaves three comments pinned directly to the files.
5. Freelancer sees context — Nadia is notified of the comments and replies in the same thread, with full context of what each comment refers to.

**Failure/Recovery Variant:** The upload is interrupted mid-transfer when Nadia's connection drops. The upload resumes automatically from where it left off rather than restarting, and Priya is not notified until the complete, correct file is actually ready.

**Success Outcome:** The client's feedback is specific, pinned to the right files, and visible to Nadia in one thread — no WhatsApp screenshots involved.

**Connected Features:** Milestone & Payment Schedule Setup, Deliverable Upload & Sharing, Large File Handling & Storage, Deliverable Review & Feedback, Deliverable Version History

---

### Approve a Milestone and Auto-Issue the Next Invoice

**Owning Persona:** Owen

**Coverage:** Regular

**Journey Goal:** Approve a completed milestone and have the next invoice issue automatically, with a permanent record of the decision.

**Entry Point:** Owen has reviewed the deliverable and any feedback from Priya and is satisfied with the work.

**Steps:**

1. Review the round — Owen opens the milestone, reviews the deliverable and the existing comment thread.
2. Approve — He clicks Approve. The action is timestamped and cannot be reversed by him afterward.
3. Invoice issues automatically — The next invoice in the payment schedule generates and is emailed to Owen immediately, with no action from Nadia.
4. Record is written — The approval and the resulting invoice are both written to the project's permanent activity trail.

**Failure/Recovery Variant:** Owen approves too early, before noticing a comment Priya left minutes earlier. Nadia can reopen the milestone — itself a logged, non-silent event distinct from the original approval — so the record stays honest about what actually happened.

**Success Outcome:** The milestone is approved, the correct invoice has already been sent without Nadia lifting a finger, and both actions are permanently on the record.

**Connected Features:** Milestone Approval, Invoice Generation & Sending, Immutable Activity & Audit Trail

---

### Chasing an Overdue Invoice — Automatically

**Owning Persona:** Nadia, Owen

**Coverage:** Edge/Recovery

**Journey Goal:** Get an overdue invoice paid without the freelancer having to manually chase it.

**Entry Point:** An invoice's due date passes without payment.

**Steps:**

1. Day 3 reminder — A polite reminder email goes to Owen automatically, three days after the due date, with no action from Nadia.
2. Still unpaid — The invoice remains open; Nadia sees its Overdue status on her dashboard without needing to check manually.
3. Day 10 reminder — A second reminder goes out automatically at day 10.
4. Payment arrives — Owen pays from the reminder's link; the invoice updates to Paid immediately and the reminder schedule stops. If he had still not paid after day 10, Nadia could send a one-click manual reminder instead of writing an email. [AUDIT-ADDED: 1 -- next step after the last automatic reminder]

**Failure/Recovery Variant:** Owen is genuinely disputing the amount rather than simply forgetting. Nadia pauses reminders for that one invoice while the conversation happens outside the schedule, without affecting reminders on any other invoice.

**Success Outcome:** The invoice gets paid with zero manual chasing from Nadia, and the record shows exactly when each reminder went out and when payment landed.

**Connected Features:** Automated Payment Reminders, Invoice Payment Processing, Immutable Activity & Audit Trail

---

### Month-End Financial Review

**Owning Persona:** Nadia

**Coverage:** Regular

**Journey Goal:** See exactly what has been earned, what is outstanding, and what is overdue across all clients, and pull a file for bookkeeping.

**Entry Point:** Nadia opens Clientroom at month end to check her overall financial position.

**Steps:**

1. Open the dashboard — She sees aggregate earned, outstanding, and overdue totals across every client at a glance.
2. Drill into a client — She opens one client's detail to see which specific invoices are behind.
3. Export for bookkeeping — She generates a CSV export of the month's invoices and payments.
4. Download and import — She downloads the file and imports it into her own accounting software.

**Failure/Recovery Variant:** The selected export date range has no invoices in it (a quiet month for one client). The export clearly states there is nothing to export for that range rather than producing a broken empty file.

**Success Outcome:** Nadia has a clear, accurate financial picture across her whole practice and a usable file for her own books, in a few minutes.

**Connected Features:** Freelancer Financial Dashboard, Accounting Export

---

### Growing Past the Free Tier

**Owning Persona:** Nadia

**Coverage:** Regular

**Journey Goal:** Add a new client beyond the free tier's limit and continue working without interruption.

**Entry Point:** Nadia, currently on the free tier with two clients, signs a third client.

**Steps:**

1. Attempt to add a client — She tries to add her third active client.
2. Upgrade prompt — She is prompted that adding this client requires the paid plan.
3. Subscribe — She subscribes at the flat monthly or yearly price.
4. Continue working — The new client is added immediately and she proceeds exactly as before, with no disruption to her existing clients.

**Failure/Recovery Variant:** Her subscription charge fails (an expired card). She sees the specific reason and can retry with a different payment method immediately; her existing clients and their data remain fully accessible throughout — nothing is locked mid-session over a billing hiccup.

**Success Outcome:** Nadia's growing client roster is reflected in her plan without any interruption to the clients she already serves.

**Connected Features:** Subscription Plan & Billing Management, Client & Project Management

---

### Client's First Login and First Feedback

**Owning Persona:** Priya

**Coverage:** First-use

**Journey Goal:** Get from a first invitation email to leaving useful feedback on a deliverable, with no account to set up.

**Entry Point:** Owen, the Primary Contact at her company, invites Priya as a Reviewer contact so she can weigh in on design work.

**Steps:**

1. Invitation received — Priya gets an email inviting her as a Reviewer contact for her company's project with Nadia.
2. First sign-in — She requests a magic link and opens the portal directly in her scoped view — no password, no account form.
3. See what's there — She sees the company's current project and any deliverables ready for review.
4. Leave feedback — She opens a deliverable and leaves a comment pinned to it.

**Failure/Recovery Variant:** Priya tries to click an Approve control she does not have, expecting Reviewer access to include it. The portal clearly shows her role's scope (comment only) rather than a confusing broken button, and she is not blocked from leaving feedback because of it.

**Success Outcome:** Priya reaches her company's project and leaves specific, useful feedback within minutes of her first invitation, with zero setup friction.

**Connected Features:** Client Contact Management & Roles, Client Portal Access (Magic-Link Login), Deliverable Review & Feedback

---

### Pointing to the Record in a Scope Dispute

**Owning Persona:** Nadia

**Coverage:** Edge/Recovery

**Journey Goal:** Resolve a disagreement about what was agreed by pointing to the permanent approval record, and handle any resulting refund or cancellation cleanly.

**Entry Point:** A client claims a piece of work was never approved, disputing an invoice Nadia already sent.

**Steps:**

1. Open the trail — Nadia opens the project's activity trail and finds the exact timestamped approval in question.
2. Show the record — She shares a printable copy of the recorded approval, which cannot have been silently altered, settling the factual question of what happened. [AUDIT-ADDED: 1 -- the freelancer needs something to show the client, not only to look at]
3. Decide the outcome — If the dispute is resolved in the client's favor, she issues a refund through her own payment processor.
4. Record the outcome — She marks the corresponding invoice Refunded in Clientroom (and the project Cancelled, if work has stopped), preserving rather than deleting the original record.

**Failure/Recovery Variant:** The client insists a milestone was never actually shown to them. The activity trail's timestamped entry for the deliverable upload and the client's own first-view timestamp settle the disagreement with evidence rather than argument. [MODIFIED: synthesis check — these two entries are now explicitly recorded by Immutable Activity & Audit Trail; the draft trail recorded only acceptances, approvals, and sent invoices]

**Success Outcome:** The dispute is resolved using the product's own permanent record, and the outcome — refund or cancellation — is itself recorded truthfully rather than erasing what came before.

**Connected Features:** Immutable Activity & Audit Trail, Refund & Cancelled Project Handling, Deliverable Upload & Sharing, Client Portal Access (Magic-Link Login) [MODIFIED: synthesis check — the failure variant relies on upload and client-view records, so their source features are listed]

---

### Exporting Data and Closing the Account

**Owning Persona:** Nadia

**Coverage:** Edge/Recovery

**Journey Goal:** Leave the product entirely, with all of her data in hand and nothing left behind that she did not intend to keep.

**Entry Point:** Nadia decides to stop using Clientroom and wants her data and a clean account closure.

**Steps:**

1. Adjust notification preferences first — On her way out, she reviews Settings and turns off optional notifications she no longer wants.
2. Request a full data export — She requests an export of every client, project, proposal, invoice, and activity record she owns.
3. Download the archive — The export completes and she downloads the full archive.
4. Request account deletion — She separately requests account deletion, confirming explicitly given its irreversibility.

**Failure/Recovery Variant:** She has an active unpaid invoice at the time of deletion. Rather than silently deleting it or blocking her outright, the product warns her specifically about the open invoice and lets her decide how to proceed before finalizing deletion.

**Success Outcome:** Nadia leaves with a complete copy of her own data, and her account and its data are permanently and correctly removed once she confirms.

**Connected Features:** Settings & Account Management, Data Export & Account Deletion

---

### Getting Help Without Giving Up Control

**Owning Persona:** Nadia, Dana

**Coverage:** Edge/Recovery

**Journey Goal:** Get a confusing account problem fixed quickly by the Clientroom operator, without handing anyone the ability to change her records.

**Entry Point:** Owen tells Nadia that the pay link on his latest invoice says online payment is temporarily unavailable, and Nadia cannot see why. [AUDIT-ADDED: 3 -- journey added so the Support Operator role and Operator Support Access are exercised end to end]

**Steps:**

1. Something looks wrong — Nadia sees a "needs attention" notice on her payment connection but does not understand what the processor is asking for.
2. Contact support — She sends a support request from inside Clientroom describing the problem and gets an email confirming it was received.
3. Read-only look — Dana opens a read-only support session on Nadia's account; Nadia receives an email notice that a support session has started.
4. Guided fix — Dana sees the processor's request for more account details and replies by email explaining exactly what to do; Nadia reconnects her payment account herself.
5. Record of the visit — The support session appears in Nadia's activity trail with its start and end time, and Owen's pay link works again.

**Failure/Recovery Variant:** Dana spots a second problem — a client contact's email address has a typo, so invitations bounce — but she cannot fix it, because nothing can be changed in a support session. She tells Nadia which contact to correct; Nadia fixes it in a few seconds, and the record shows only Nadia's own change.

**Success Outcome:** Nadia's payments work again within one exchange with support, she knows exactly who looked at her account and when, and nothing in her records was changed by anyone but her.

**Connected Features:** Operator Support Access, Payment Account Connection, Immutable Activity & Audit Trail, Notifications (Email), Client Contact Management & Roles
