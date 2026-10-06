# Session notes — Stage 1 intake (n2b:s1-init)

Every question n2b asked during Stage 1 and the answer given as the founder (from `napkin.md` and
`ANSWERS.md`). Run date: 2026-09-26. The run was unattended, so the founder's answers came from those two files.

### Q: What do you want to build? Tell me everything — the idea, the problem, who it's for, how you imagine it working. (open capture)

The full body of `napkin.md`, verbatim (also saved at `.n2b/inputs/source/pasted-notes.md`):

```text
I'm a solo founder and a freelance product designer. I want to build a client portal for
freelancers — designers, developers, copywriters, video editors — that replaces the mess of
email threads, shared drives, and invoice spreadsheets. Working title: Clientroom.

THE PROBLEM
Every freelance project runs across five tools that don't talk to each other. The proposal
is a PDF in email, the files are in a Google Drive folder with 40 versions called
"final_v3_REAL", feedback arrives as a screenshot on WhatsApp from someone who wasn't on
the original thread, and invoices live in a spreadsheet plus a payment link I paste into
an email and then chase for six weeks. Clients never know what stage we're at, I never
know which "approved" counts, and when a client disputes scope there's no record of what
they signed off. The tools that exist are either full agency project-management suites
(heavy, per-seat pricing, built for teams of 20), or invoicing apps that don't know
anything about the project. Lots of freelancers try to build this in Notion and it falls
apart the first time a client needs to approve something.

WHAT IT IS
A branded client portal per freelancer. For each client project there's one link where the
client sees the agreed proposal, the milestones and where we are, every deliverable with
its version history, a clear "approve" or "request changes" on each, and every invoice
with a way to pay by card or bank transfer. The freelancer sees all their clients, all
their projects, and all their money — what's paid, what's due, what's overdue — in one
place, and the reminders and the paper trail happen automatically.

WHO USES IT
1. The freelancer (the owner). Creates clients and projects, writes proposals, sets
   milestones and payment schedules (deposit up front, per milestone, or on completion),
   uploads deliverables, sends invoices, sees everything.
2. Client contacts. A client is usually a small company with more than one person
   involved: the person who signs and pays, and one or two people who give feedback. Not
   everyone should be able to approve work or see invoices — I'm not sure exactly how to
   split that (see open questions). Client contacts only ever see their own company's
   projects, and they should get in without creating yet another password.
Solo freelancers only for v1. Agencies with several team members are a different product,
maybe later. I as the operator only need basic support access, nothing more.

THE EXPERIENCE
I send a proposal to a new client from Clientroom. Their founder opens the link, reads the
scope and price, and clicks "Accept" — that's recorded with a timestamp and a deposit
invoice appears, which they pay by card on the spot. Two weeks later I upload the first
round of designs to milestone 2. The client's marketing lead gets an email, opens the
portal on their phone, leaves three comments pinned to the files, and the founder hits
"Approve" the next day. Approving the milestone automatically issues the next invoice.
When an invoice goes overdue, polite reminders go out on day 3 and day 10 without me
typing a word. At month end I open my dashboard: earned, outstanding, overdue, per client.
The key moment is the first time a client says "I love that everything is in one place"
and pays within a day.

BUSINESS
Subscription per freelancer, priced by number of active clients: a free tier for one or
two clients, then a flat monthly or yearly price. No cut of payments — freelancers hate
platform fees on top of processor fees. Payments go straight to the freelancer's own
account; I never hold their money. Why now: freelancing keeps growing, clients now expect
a "portal" experience, and the existing tools are built for agencies, not individuals.
First users: my own freelance network and design communities.

SCALE & ENVIRONMENT
First year: a few thousand freelancers, each with 3 to 15 active clients and a handful of
contacts per client. Files are big (design files, videos, PDFs), typically tens of MB,
sometimes over a GB for video. Freelancers work on laptops; clients often review on their
phones, so the client side must be great on mobile. Freelancers and clients are all over
the world: currencies, tax on invoices (VAT, GST, US sales tax) and time zones must not be
hard-coded. Start with English only.

WHAT IT MUST LIVE ALONGSIDE
- Card and bank payments through an established payment processor, paid directly into the
  freelancer's own account. I never want to touch card numbers.
- Email for all notifications (clients won't install an app).
- Accounting software (QuickBooks, Xero) — freelancers will ask; export is fine for v1.
- Figma, Google Drive, Dropbox links as deliverables — link, don't copy, for v1.
- Each freelancer's own logo, colours and ideally their own domain on the portal.
Otherwise it stands alone.

ONE YEAR FROM NOW, IT WORKED IF
- Freelancers get paid noticeably faster and chase invoices far less.
- Clients stop sending feedback by WhatsApp and screenshot.
- When a scope argument happens, the freelancer can point at the approval record.
- Most new freelancers arrive because a client or peer saw someone else's portal.

HARD BOUNDARIES
- Team: just me, building with AI coding tools. First paying freelancer within about
  three months.
- Budget: infrastructure under roughly $100/month until there's revenue — file storage and
  bandwidth for big files has to be thought through.
- Money: payments go directly to the freelancer through the processor; the platform never
  holds or moves funds and never stores card data.
- Records: accepted proposals, approvals and sent invoices must never be silently edited
  afterwards — they are the freelancer's evidence.
- Privacy: a client can never see another client's anything; freelancers must be able to
  export and delete their data.
- Platform: web app for v1 (freelancer on desktop, clients on mobile browser). No native
  apps.
- No technology commitments — build it with whatever the blueprint recommends.

THINGS I'M NOT SURE ABOUT — please record these as open questions rather than guess
- Client contact roles: who can approve, who can see invoices, who can invite colleagues?
- Should proposals be legally binding e-signatures, or is a recorded "Accept" enough?
- Is time tracking in scope, or does that belong in another tool?
- How far should tax handling go — just showing the tax line, or calculating it per
  country?
- Custom domain per freelancer in v1, or later?
```

### Q: You said not everyone at the client company should approve work or see invoices. How do you read the split between client contacts? (interpretation check, batched)

Open question. Leaning: a "primary" contact who can accept proposals, approve milestones and see/pay invoices, and "reviewer" contacts who can comment only. Who can invite colleagues is also still open.

### Q: Clients "get in without creating yet another password". Does that mean a magic link emailed to them? (interpretation check, batched)

Yes. Passwordless: a magic link by email. No account-creation wall.

### Q: Payment schedules are deposit, per milestone or on completion. Can a schedule mix those, can clients pay part of an invoice, and are the day-3 / day-10 reminders fixed? (interpretation check, batched)

A schedule is set per project and can be deposit, per milestone, on completion, or a mix. Invoices are paid by card or bank transfer through the processor, straight into the freelancer's own account. The platform takes no cut and never holds funds. Partial payments are allowed. Overdue reminders are automatic on day 3 and day 10 by default, and the freelancer can change that.

### Q: You as the operator need "basic support access". Is that read-only access to a freelancer's account rather than a product role? (interpretation check, batched)

Yes. Read-only support access to a freelancer's account. It is not a product role.

### Q: Who is the very first freelancer you picture using this, and what makes each side open it?

A solo product designer like me with 5–8 active clients, currently juggling email, Google Drive, WhatsApp feedback and an invoice spreadsheet. The freelancer opens it to send a proposal, upload a deliverable, or check the money at month end. The client opens it from an email about a new deliverable, an approval request or an invoice.

### Q: If a client has paid and something goes wrong (a cancelled project, a disputed invoice), how does money get back to them?

(ANSWERS.md is silent, so I gave the conventional answer.) Any refund is issued by the freelancer through the payment processor from their own account. The platform never moves the money. The exact refund and dispute flow is recorded as an open question.

### Q: You mentioned each freelancer's logo, colours and maybe their own domain. Do you have design artifacts to supply (files, a URL, design tokens)?

No design system supplied and no artifacts to ingest. Preference only: clean, professional, lots of white space. Each freelancer sets their logo and brand colour, and maybe later a custom domain.

### Q: Money, tax and personal data all show up here. To confirm the regulated areas: card payments (held by the processor), tax on invoices, personal-data export and deletion, and record immutability. Anything else?

Yes to all of those: payments (card data owned by the processor), tax on invoices, personal-data export and deletion (GDPR, since clients are worldwide), and record immutability. No other regulated data.

### Q: Before I play this back: any hard boundaries I should know about? Timeline, budget, regulations, existing systems it must work with, technology commitments, brand rules, anything non-negotiable? (the constraints question, asked once)

The HARD BOUNDARIES list from the napkin, nothing else:
- Team: just me, building with AI coding tools. First paying freelancer within about three months.
- Budget: infrastructure under roughly $100/month until there's revenue. File storage and bandwidth for big files have to be thought through.
- Money: payments go directly to the freelancer through the processor. The platform never holds or moves funds and never stores card data.
- Records: accepted proposals, approvals and sent invoices must never be silently edited afterwards. They are the freelancer's evidence.
- Privacy: a client can never see another client's anything. Freelancers must be able to export and delete their data.
- Platform: web app for v1 (freelancer on desktop, clients on mobile browser). No native apps.
- No technology commitments. Build it with whatever the blueprint recommends.

### Q: How would you like to proceed? (fork after the show-back)

**Looks good — take it from here** (Path C). Feature discussion was declined so the repo shows n2b's own feature discovery.

### Q: Which AI models should n2b's agents use? (pipeline settings)

**Balanced — smart planning, fast execution (Recommended).** Provider: Claude aliases (the default, not asked on Claude Code).
