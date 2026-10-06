# Stage 1 answer key — Clientroom

You are the founder. Everything in `napkin.md` counts as *given*. If n2b still asks, answer from the
table below in the founder's voice. Don't volunteer more than asked. Where this is silent, choose the
conventional option and let n2b record it as an open question.

## Fixed choices

| Prompt | Choice |
|---|---|
| Open capture ("tell me about your idea") | The full body of `napkin.md`, verbatim |
| Fork after the show-back | **Looks good — take it from here** (Path C). Decline feature discussion (Path D). |
| Model profile | **balanced** |
| Provider | **Claude aliases** (default) |
| Design-system artifacts? | None. Blueprint ships design-agnostic; the preference below goes into Constraints. |
| Existing `.n2b/` found | Should not happen in a fresh run — if it does, stop and report. |

## Follow-up answers

| If it asks about… | Answer |
|---|---|
| Product in two sentences | "A branded client portal for solo freelancers: proposals, milestones, deliverables with approvals, and invoices in one link per project. Clients know where things stand and pay faster; the freelancer keeps a clean paper trail." |
| Who the *first* user is | A solo product designer like me with 5–8 active clients, currently juggling email, Google Drive, WhatsApp feedback and an invoice spreadsheet. |
| Trigger for usage | Freelancer: sending a proposal, uploading a deliverable, month-end money check. Client: email notification of a new deliverable, approval request or invoice. |
| Roles confirmed? | Yes: freelancer (owner) and client contacts, several per client company. Exact contact permissions are an open question. Solo freelancers only in v1 — agencies out of scope. |
| Operator / support role | Read-only support access to a freelancer's account; not a product role. |
| Client contact permissions | Open question. Leaning: a "primary" contact who can accept proposals, approve milestones and see/pay invoices, and "reviewer" contacts who can comment only. |
| Client sign-in | Passwordless: magic link by email. No account creation wall. |
| Money flow | Payment schedule set per project (deposit, per milestone, on completion, or mixed). Invoices paid by card or bank transfer through the processor, straight into the freelancer's own account. Platform takes no cut and never holds funds. Partial payments allowed. Overdue reminders automatic (day 3 and day 10, configurable). |
| Approvals | Each deliverable version can be approved or sent back with comments; milestone approval can trigger the next invoice. Approvals and accepted proposals are timestamped and immutable. |
| Proposals | Accept button with a recorded, timestamped acceptance; whether a legally binding e-signature is needed is an open question. |
| Deliverable files | Upload files (up to ~1 GB for video) with version history, or link Figma/Drive/Dropbox. Storage cost must be managed within budget. |
| How it makes money | Subscription per freelancer by number of active clients: free for 1–2 clients, then one flat monthly/yearly price. No payment fees. |
| Why now | Freelancing keeps growing; clients expect a portal; existing tools are agency-sized or invoice-only. |
| Order of magnitude | A few thousand freelancers year one; 3–15 active clients each; a handful of contacts per client. |
| Devices / platforms | Web app: freelancer on desktop, clients mostly on mobile browsers. No native apps. |
| Geography | Worldwide from day one: multi-currency, tax lines (VAT/GST/sales tax), time zones; English only. |
| Availability / performance | Payments and records must be correct and never altered; no specific uptime number. |
| Integrations | Payment processor (required), email (required), accounting export (CSV/QuickBooks/Xero-compatible file) for v1, Figma/Drive/Dropbox as links. |
| Regulated-domain confirmation | Yes: payments (processor-owned card data), tax on invoices, personal data export/deletion (GDPR since clients are worldwide), record immutability. No other regulated data. |
| Design preferences / brand | No design system supplied. Preference only: clean, professional, lots of white space; each freelancer sets logo, brand colour and (maybe later) a custom domain. No artifacts to ingest. |
| Constraints question ("anything non-negotiable?") | Repeat the HARD BOUNDARIES list; nothing else. |
| Feature discussion (Path D) | Decline for this showcase — pick Path C so the repo shows n2b's own discovery. |
