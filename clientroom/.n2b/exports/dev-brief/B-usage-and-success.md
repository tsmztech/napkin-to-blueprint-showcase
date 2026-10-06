# Part B — Usage & Success

This part shows how the product is used end to end and how success is measured. Journeys double as integration-test narratives; metrics carry the testable targets.

## How It's Used

The user journeys follow.


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


## What Success Looks Like

The success metrics follow.


# Success Metrics

## Summary

This document contains 21 success metrics covering all 17 Core features plus the Important features that carry the brief's growth and business goals (Portal Referral Attribution, Onboarding / First-Run Setup, Subscription Plan & Billing Management): 9 product-experience metrics, 8 adoption/engagement/business KPIs, and 4 user-facing performance expectations. Together they validate the brief's Success Criteria: freelancers get paid faster and chase less, clients stop using WhatsApp screenshots, disputes are settled by evidence, and growth arrives through referral. [MODIFIED: 3 metrics added — Payment Readiness Before First Invoice for the new Core feature Payment Account Connection, First-Session Activation for the research-backed setup-speed goal, and Free-to-Paid Conversion for the brief's subscription business model]

---

### Client and Project Setup Speed

**Description:** Measures how quickly Nadia can add a new client and start a project, since this is the first action in every new engagement.

**Target:** Nadia can add a client and create a project under it in under 2 minutes, from opening the "add client" action to the project appearing in her roster.

**Rationale:** BRIEF.md's Vision promises "one link per client project"; if setup itself is slow, the product fails its own premise before any proposal is even drafted.

**Persona:** Nadia

**Connected Feature:** Client & Project Management

---

### Proposal Send Speed

**Description:** Measures how quickly Nadia can go from a blank project to a sent proposal.

**Target:** Nadia can draft and send a proposal in under 10 minutes for a typical project scope.

**Rationale:** BRIEF.md's Problem Statement describes proposals today as a slow, email-bound PDF process; a fast, structured alternative is the product's first value moment for a new client relationship.

**Persona:** Nadia

**Connected Feature:** Proposal Creation & Sending

---

### Time to Proposal Acceptance

**Description:** Measures how quickly a client accepts a proposal once it is sent, an indicator of how frictionless the acceptance flow is.

**Target:** At least 70% of accepted proposals are accepted within 24 hours of being sent.

**Rationale:** BRIEF.md's Experience narrative describes acceptance happening the moment the client opens the link ("reads the scope and price, and clicks 'Accept'"); a fast acceptance rate confirms the flow removes friction rather than adding it.

**Persona:** Owen

**Connected Feature:** Proposal Acceptance

---

### Milestone Schedule Completeness

**Description:** Measures whether every accepted project has a complete, unambiguous payment schedule before work begins.

**Target:** 100% of projects with an accepted proposal have at least one milestone with an assigned payment trigger before the first deliverable is uploaded.

**Rationale:** BRIEF.md's Success Criteria depends on a defensible record of what was agreed; an incomplete schedule undermines both invoicing and the evidence trail from the start.

**Persona:** Nadia

**Connected Feature:** Milestone & Payment Schedule Setup

---

### Client Portal Login Success

**Description:** Measures whether client contacts can reliably reach their scoped portal view via the magic-link flow, since this gates every client-facing action.

**Target:** At least 95% of magic-link sign-in attempts succeed on the first try, and a failed or expired attempt can be recovered with one additional request.

**Rationale:** BRIEF.md, Target Users & Roles requires that contacts "never hit an account-creation wall"; a low success rate here would silently block every downstream client action.

**Persona:** Owen

**Connected Feature:** Client Portal Access (Magic-Link Login)

---

### Deliverable Upload Reliability

**Description:** Measures whether large deliverable uploads complete successfully, including after an interrupted connection.

**Target:** At least 98% of deliverable uploads, including files over 500 MB, complete successfully without the user needing to restart from zero.

**Rationale:** BRIEF.md, Scale & Non-Functional Expectations calls out files "sometimes over 1 GB for video" as normal; an unreliable upload path would break the product's core delivery mechanism for its stated user base.

**Persona:** Nadia

**Connected Feature:** Deliverable Upload & Sharing

---

### Feedback Consolidation

**Description:** Measures whether client feedback actually moves into pinned, contextual comments rather than staying in outside channels like WhatsApp.

**Target:** Within 3 months of a freelancer's first client onboarding to the portal, at least 80% of that client's feedback on deliverables arrives as in-portal comments rather than through outside channels the freelancer reports separately.

**Rationale:** BRIEF.md's Success Criteria states plainly: "Clients stop sending feedback by WhatsApp and screenshot." This metric is the most direct test of that outcome.

**Persona:** Priya

**Connected Feature:** Deliverable Review & Feedback

---

### Milestone Approval Turnaround

**Description:** Measures how quickly a client approves a milestone once a deliverable and any requested revisions are ready.

**Target:** At least 60% of milestones are approved within 5 days of the deliverable being marked ready for final review.

**Rationale:** BRIEF.md's Experience narrative treats approval as the trigger for automatic invoicing; a slow approval turnaround directly delays the freelancer getting paid, undermining the product's core promise.

**Persona:** Owen

**Connected Feature:** Milestone Approval

---

### Invoice Auto-Generation Accuracy

**Description:** Measures whether automatically generated invoices carry the correct amount, currency, and tax line every time.

**Target:** 100% of automatically generated invoices match their triggering milestone, deposit, or completion amount, with no manual correction needed.

**Rationale:** BRIEF.md's Constraints demand that "payments and records must be correct"; an incorrect auto-generated invoice would directly damage the trust the product is built to create.

**Persona:** Nadia

**Connected Feature:** Invoice Generation & Sending

---

### Time to Payment

**Description:** Measures how quickly invoices are paid after being sent, the product's headline outcome.

**Target:** The median invoice is paid within 3 days of being sent, down from the multi-week chase described in BRIEF.md's Problem Statement.

**Rationale:** BRIEF.md's Success Criteria: "Freelancers get paid noticeably faster." This is the single clearest measure of whether the product delivers its central promise.

**Persona:** Nadia

**Connected Feature:** Invoice Payment Processing

---

### Reminder-Driven Payment Recovery

**Description:** Measures how often automated reminders alone (without the freelancer manually following up) bring in payment on an overdue invoice.

**Target:** At least 50% of invoices that go overdue are paid after an automated reminder and before any manual follow-up from the freelancer.

**Rationale:** BRIEF.md's Vision states reminders and record-keeping "happen automatically, so the freelancer stops chasing." This metric directly measures whether automation is doing that job.

**Persona:** Nadia

**Connected Feature:** Automated Payment Reminders

---

### Dashboard Comprehension

**Description:** Measures whether the freelancer can understand her overall financial position from the dashboard without extra explanation or manual calculation.

**Target:** Nadia can state her total earned, outstanding, and overdue amounts across all clients within 10 seconds of opening the dashboard.

**Rationale:** BRIEF.md's Experience narrative: "you open your dashboard and see earned, outstanding and overdue, per client." This is the exact moment the metric validates.

**Persona:** Nadia

**Connected Feature:** Freelancer Financial Dashboard

---

### Dispute Resolution Confidence

**Description:** Measures whether the activity trail actually gives freelancers something concrete to point to when a scope disagreement happens.

**Target:** In at least 90% of reported scope disputes, the freelancer is able to locate a specific, relevant timestamped record in the activity trail within 2 minutes.

**Rationale:** BRIEF.md's Success Criteria states: "When a scope argument happens, the freelancer can point at the approval record." This metric tests whether the record is actually usable in the moment it matters.

**Persona:** Nadia

**Connected Feature:** Immutable Activity & Audit Trail

---

### Notification Delivery Reliability

**Description:** Measures whether event-triggered emails (proposals, deliverables, approvals, invoices, reminders) actually reach recipients.

**Target:** At least 99% of triggered notifications are successfully delivered, with any failure surfaced to the freelancer within minutes rather than silently lost; fewer than 1 in 100 client contacts report finding a Clientroom email in their spam folder. [RESEARCH-INFORMED: spam-folder delivery of client emails is a recurring complaint for HoneyBook and SuiteDash, and message reliability is a category-wide quality bar (G2, Capterra, Trustpilot, HIGH)]

**Rationale:** BRIEF.md, Ecosystem & Integrations: "Clients will not install an app," making email the sole channel reaching them; undelivered email breaks every client-facing workflow at once.

**Persona:** All

**Connected Feature:** Notifications (Email)

---

### Invoice Currency and Tax Correctness

**Description:** Measures whether invoices consistently show the correct currency and an appropriate tax line for the client's configured region.

**Target:** 100% of invoices display the project's configured currency and tax line correctly, across every currency and tax configuration in active use.

**Rationale:** BRIEF.md, Scale & Non-Functional Expectations requires that "currencies, tax on invoices... must not be hard-coded," since the product serves clients worldwide from day one.

**Persona:** Nadia

**Connected Feature:** Currency & Tax Handling

---

### Large File Upload Success at Scale

**Description:** Measures whether large-file handling stays reliable as a freelancer's total stored deliverables grow, without breaking the stated infrastructure budget.

**Target:** Upload and download performance for files up to 1 GB+ remains consistent (no more than a brief, visible delay) as a freelancer's account approaches typical year-one storage volumes.

**Rationale:** BRIEF.md's Constraints bound infrastructure spend to roughly $100/month; this metric is the user-observable signal that the budget constraint has not degraded the deliverable experience.

**Persona:** Nadia

**Connected Feature:** Large File Handling & Storage

---

### Growth Through Referral

**Description:** Measures whether new freelancers are arriving because a client or peer saw someone else's branded portal, the product's intended growth loop.

**Target:** At least a third of new freelancer signups in year one cite seeing another freelancer's client portal (as a client or peer) as how they heard about the product.

**Rationale:** BRIEF.md's Business Context and Success Criteria both name this growth loop explicitly: "Most new freelancers arrive because a client or peer saw someone else's portal." Portal-referred sign-ups are counted from the referral mark and the "how did you hear" answer.

**Persona:** All

**Connected Feature:** Portal Referral Attribution [MODIFIED: synthesis check — re-pointed from Freelancer Branding to Portal Referral Attribution, the feature added to deliver and measure this growth loop]

---

### Client Portal Mobile Responsiveness

**Description:** Measures whether the client-facing portal feels fast and native on mobile browsers, since clients review almost entirely on their phones.

**Target:** Client-facing pages (deliverable review, approval, invoice payment) load and become interactive within 2 seconds on a typical mobile connection.

**Rationale:** BRIEF.md, Scale & Non-Functional Expectations: "clients mostly review on mobile browsers, so the client side must be excellent on mobile." A sluggish mobile experience would undermine every client-facing feature at once.

**Persona:** Owen

**Connected Feature:** Client Portal Access (Magic-Link Login)

---

### Payment Readiness Before First Invoice

**Description:** Measures whether freelancers have their own payment account connected by the time their first invoice goes out, so the client can pay on the spot. [AUDIT-ADDED: 1 -- metric for the Core feature added by the value-flow walk]

**Target:** At least 90% of freelancers have a connected, ready payment account at the moment their first invoice is sent, and connecting takes under 5 minutes of the freelancer's own time.

**Rationale:** BRIEF.md's Experience narrative has the client paying the deposit "by card on the spot"; that moment only exists if the freelancer's own account is connected first (BRIEF.md, Ecosystem & Integrations: payment processor required).

**Persona:** Nadia

**Connected Feature:** Payment Account Connection

---

### First-Session Activation

**Description:** Measures whether a new freelancer reaches a ready-to-send first proposal in her first session, without a long setup phase. [RESEARCH-INFORMED: setup burden of 15–25 hours (Dubsado) and 20+ hours (SuiteDash) is the category's most consistent complaint (independent reviews, HIGH)]

**Target:** At least 70% of new freelancers have a first client, project, and drafted proposal within their first session, and the median time from sign-up to that point is under 15 minutes.

**Rationale:** BRIEF.md's three-month goal of a first paying freelancer depends on new accounts reaching value immediately; a product that avoids the category's setup burden should prove it here.

**Persona:** Nadia

**Connected Feature:** Onboarding / First-Run Setup

---

### Free-to-Paid Conversion

**Description:** Measures whether freelancers who outgrow the free tier (one or two active clients) choose to pay rather than leave or stall. [AUDIT-ADDED: 4 -- monetization concern: the brief's subscription model had no business KPI]

**Target:** At least 40% of freelancers who try to add a client beyond the free limit are on a paid plan within 14 days.

**Rationale:** BRIEF.md, Business Context: Clientroom is "sold as a subscription per freelancer, priced by number of active clients: free for one or two clients." Research shows every profiled competitor uses a time-limited trial instead of a free tier (vendor pricing pages, HIGH), so this metric is the direct test of whether the brief's free-tier model converts. [CHALLENGED: trial-only is the category norm (5 sources, HIGH confidence) -- original free-tier model retained per SYN-04 protection (user-stated pricing model); this metric makes its performance visible]

**Persona:** Nadia

**Connected Feature:** Subscription Plan & Billing Management
