---
document_type: success-metrics
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed (1 fix applied)
---

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
