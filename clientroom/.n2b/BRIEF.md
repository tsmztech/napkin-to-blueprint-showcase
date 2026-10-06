---
project_name: Clientroom
domain: freelance client project delivery and getting paid
created: 2026-09-26
status: active
n2b_version: 0.4.0
---

# Clientroom

## Vision
Clientroom is a branded client portal for solo freelancers (designers, developers, copywriters, video editors). It puts proposals, milestones, deliverables with approvals, and invoices in one link per client project. Clients always know where things stand and pay faster. The freelancer sees all their clients, projects and money in one place, and keeps a clean, unalterable paper trail of what was agreed, approved and billed. Reminders and record-keeping happen automatically, so the freelancer stops chasing.

## Problem Statement
Every freelance project today runs across about five tools that don't talk to each other. The proposal is a PDF in email. Files sit in a shared drive folder full of "final_v3_REAL" versions. Feedback arrives as WhatsApp screenshots from people who weren't on the thread. Invoices live in a spreadsheet plus a pasted payment link that gets chased for weeks. Clients never know what stage the project is at, the freelancer never knows which "approved" counts, and when scope is disputed there is no record of what was signed off. The existing tools don't fit: agency project-management suites are heavy, priced per seat and built for teams of 20; invoicing apps know nothing about the project; home-made Notion setups fall apart the first time a client has to approve something.

## Target Users & Roles
Roles were confirmed in conversation. Solo freelancers only for v1. Agencies with several team members are out of scope.

- **Freelancer (owner).** First user: a solo product designer with 5–8 active clients, currently juggling email, Google Drive, WhatsApp feedback and an invoice spreadsheet. The freelancer creates clients and projects, writes and sends proposals, sets milestones and payment schedules (deposit, per milestone, on completion, or a mix), uploads or links deliverables, sends invoices, and sees every client, project and payment: paid, due and overdue. They open it to send a proposal, upload a deliverable, or check the money at month end.
- **Client contacts.** Several people per client company, usually a small company: the person who signs and pays, and one or two people who give feedback. Contacts only ever see their own company's projects, never another client's. They sign in passwordless with a magic link by email and never hit an account-creation wall. They open the portal from an email telling them about a new deliverable, an approval request or an invoice, often on their phone. Not every contact may approve work or see invoices. The exact permission split is an open question; the founder's leaning is a "primary" contact who can accept proposals, approve milestones and see and pay invoices, and "reviewer" contacts who can only comment.
- **Operator support access (not a product role).** The founder, as operator, needs read-only support access to a freelancer's account, nothing more.

## The Experience
You send a proposal to a new client from Clientroom. Their founder opens the link, reads the scope and price, and clicks "Accept". The acceptance is recorded with a timestamp, a deposit invoice appears, and they pay it by card on the spot. Two weeks later you upload the first round of designs to milestone 2. The client's marketing lead gets an email, opens the portal on their phone, and leaves three comments pinned to the files. The next day the founder hits "Approve", and approving the milestone automatically issues the next invoice. When an invoice goes overdue, polite reminders go out on day 3 and day 10 without you typing a word. At month end you open your dashboard and see earned, outstanding and overdue, per client. The key moment is the first time a client says "I love that everything is in one place" and pays within a day.

## Business Context
Clientroom is a commercial venture sold as a subscription per freelancer, priced by number of active clients: free for one or two clients, then one flat monthly or yearly price. There is no cut of payments, because freelancers resent platform fees on top of processor fees. Payments go straight to the freelancer's own processor account and the platform never holds their money. Why now: freelancing keeps growing, clients now expect a "portal" experience, and existing tools are built for agencies or only handle invoicing. First users come from the founder's own freelance network and design communities. The intended growth loop is that new freelancers arrive after a client or peer sees someone else's portal. Specific price points are not yet set.

## Scale & Non-Functional Expectations
- **Users (year one):** a few thousand freelancers, each with 3–15 active clients and a handful of contacts per client.
- **Data:** large files are the norm: design files, videos and PDFs, typically tens of MB and sometimes over 1 GB for video, with version history per deliverable. Storage and bandwidth cost must fit the budget constraint.
- **Devices / platforms:** web app. Freelancers work on laptop or desktop; clients mostly review on mobile browsers, so the client side must be excellent on mobile. No native apps.
- **Geography:** worldwide from day one. Currencies, tax on invoices (VAT, GST, US sales tax) and time zones must not be hard-coded. English only at launch.
- **Correctness & records:** payments and records must be correct. Accepted proposals, approvals and sent invoices are timestamped and never silently altered afterwards.
- **Privacy:** strict isolation between clients. Freelancers can export and delete their data, and personal-data handling must hold up worldwide (GDPR).
- **Availability:** no specific uptime number was stated.

## Ecosystem & Integrations
- **Payment processor (required):** an established processor takes card and bank-transfer payments directly into each freelancer's own account. The platform never touches card numbers or holds funds.
- **Email (required):** all notifications to clients and freelancers go by email. Clients will not install an app.
- **Accounting software (QuickBooks, Xero):** v1 provides an export file (CSV or a QuickBooks/Xero-compatible format). No live sync.
- **Figma, Google Drive, Dropbox:** accepted as deliverables by link, not copied into Clientroom, for v1.
- **Freelancer branding:** each freelancer's own logo and colours on their portal, and ideally their own custom domain (timing is an open question).
- Otherwise the product stands alone (confirmed).

## Success Criteria
One year from now, it worked if:
- Freelancers get paid noticeably faster and chase invoices far less.
- Clients stop sending feedback by WhatsApp and screenshot.
- When a scope argument happens, the freelancer can point at the approval record.
- Most new freelancers arrive because a client or peer saw someone else's portal.

## Constraints
- **Team / timeline:** a solo founder building with AI coding tools, with a first paying freelancer within about three months. This bounds how much can be built and operated.
- **Budget:** infrastructure under roughly $100/month until there is revenue. Storage and bandwidth for big files have to be thought through.
- **Money handling (regulatory / trust):** payments go directly to the freelancer through the processor. The platform never holds or moves funds and never stores card data.
- **Record immutability (evidence):** accepted proposals, approvals and sent invoices must never be silently edited afterwards. They are the freelancer's evidence in scope disputes.
- **Privacy (regulatory):** a client can never see another client's anything, and freelancers must be able to export and delete their data. Personal data of worldwide clients falls under GDPR.
- **Platform:** web app for v1 (freelancer on desktop, clients on mobile browser). No native apps.
- **Technology:** no technology commitments. Build with whatever the blueprint recommends.
- **Design preference (brand):** clean and professional with lots of white space. Each freelancer sets their logo and brand colour, and possibly later a custom domain. No design system or artifacts were supplied.

## Open Questions
- **Client contact roles:** who can accept proposals and approve work, who can see and pay invoices, and who can invite colleagues? The leaning is "primary" versus comment-only "reviewer" contacts, but it is not decided.
- **Proposal acceptance:** do proposals need legally binding e-signatures, or is a recorded, timestamped "Accept" enough?
- **Time tracking:** is it in scope, or does it belong in another tool?
- **Tax handling depth:** should invoices only show a tax line, or calculate tax per country or region (VAT, GST, US sales tax)?
- **Custom domain per freelancer:** in v1, or later?
- **Refunds and disputes:** refunds are assumed to be issued by the freelancer through the processor from their own account, but the refund, chargeback and cancelled-project flow was not defined.
- **Pricing:** the exact subscription price points and the free-tier limit (one or two active clients) are not set.
- **Large-file cost:** how do we store and deliver files of tens of MB up to 1 GB+ with version history within roughly $100/month of infrastructure?

## Source Materials
- `.n2b/inputs/source/pasted-notes.md`: the founder's original written brief (the problem, what it is, users, experience, business, scale, integrations, success criteria, hard boundaries and open questions), preserved verbatim.

*Preserved verbatim so nothing the user wrote is lost in this brief's compression.
Stage 2 works from this brief (brief-first); the originals are kept for reference and
for Stage 1 re-runs.*
