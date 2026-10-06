---
document_type: user-persona
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed (2 fixes applied)
---

# User Personas

## Persona Set Summary

This product serves two distinct sides of a client relationship plus one internal support role. The primary persona is the solo freelancer who owns the account; two secondary personas represent client-company contacts with different entitlements (a "primary" contact who accepts and pays, and a "reviewer" contact who only comments); a third secondary persona is the founder's own read-only support access. All four are confirmed by BRIEF.md's Target Users & Roles section. Market research surfaced one further role pattern — freelancer-side team members with scoped access, such as a bookkeeper or contractor (Dubsado's all-or-nothing permissions complaint, MEDIUM) — which is deliberately not added, because BRIEF.md limits v1 to solo freelancers with no team members. [AUDIT-EXCLUDED: 2 -- freelancer-side team and bookkeeper roles from competitor evidence excluded; BRIEF.md, Target Users & Roles: "Solo freelancers only for v1"]

## Primary Persona

### Persona Name

Nadia

### Description

Nadia is a solo product designer running her own freelance practice with 5–8 active clients at any time. She is not an agency and has no employees — every proposal, deliverable, and invoice passes through her alone. Today she juggles email for proposals, a shared drive full of versioned files, WhatsApp screenshots for feedback, and a spreadsheet plus a pasted payment link for invoicing. She sought out a product like Clientroom because the number of tools has become its own management problem, separate from the design work itself.

### Goals

- Send a proposal and get it accepted and paid (deposit) without a back-and-forth email thread
- Keep every client's deliverables, approvals, and invoices in one place per project, so she never has to reconstruct "what was agreed"
- Get paid faster and stop manually chasing overdue invoices
- See, at a glance, what she has earned, what is outstanding, and what is overdue across all clients
- Have a defensible, timestamped record of what was approved if a client later disputes scope

### Pain Points

- Feedback arrives as WhatsApp screenshots from people who were never on the original thread, so she cannot tell what has actually been agreed (BRIEF.md, Problem Statement)
- Files pile up as "final_v3_REAL" versions in a shared drive with no authoritative current version (BRIEF.md, Problem Statement)
- Invoices live in a spreadsheet with a pasted payment link that has to be manually chased for weeks (BRIEF.md, Problem Statement)
- Existing agency project-management suites are heavy, priced per seat, and built for teams of 20 — wrong shape for a practice of one (BRIEF.md, Problem Statement)
- When a scope disagreement happens, there is no record of what was actually signed off, leaving her with no way to point at evidence (BRIEF.md, Problem Statement)
- The all-in-one alternatives take days to set up before they pay off — 15–25 hours for Dubsado and 20+ hours for SuiteDash [source: independent reviews, confidence: HIGH] [RESEARCH-INFORMED: added the category's dominant complaint theme from market research]
- Tools that process payments themselves add a percentage fee on top of the subscription and sometimes hold payouts for days [source: G2 and Trustpilot reviews of HoneyBook and Bonsai, confidence: MEDIUM] [RESEARCH-INFORMED: added from market research pricing and payout complaints]

### Behavioral Context

Nadia works from a laptop or desktop throughout her working day. She opens Clientroom to send a new proposal, upload a deliverable round, check whether a client has approved or paid, or review her overall financial position at month end. Client-facing moments (sending a deliverable, an invoice) happen in bursts tied to her project schedule, not on a fixed daily cadence. She is comfortable with software but has no patience for tools built for teams of 20.

### What This User Does NOT Need

- Team seats, internal role hierarchies, or agency-style resourcing views — she works alone (BRIEF.md, Target Users & Roles: "Solo freelancers only for v1. Agencies with several team members are out of scope.")
- A platform that holds or moves her clients' money — payments go straight to her own processor account (BRIEF.md, Business Context)
- Native mobile apps for her own use — she works on laptop or desktop (BRIEF.md, Scale & Non-Functional Expectations)
- Deep project-management machinery (task boards, resourcing, time-tracking-driven billing) — the product's job is proposals, milestones, deliverables, and invoices, not full PM tooling
- A workflow or automation builder she has to configure before the product is useful — she wants sensible, fixed behavior (automatic invoices on approval, day-3 and day-10 reminders) out of the box [RESEARCH-INFORMED: setup-overhead complaint theme across Dubsado and SuiteDash, HIGH]

## Secondary Personas

### Client Primary Contact

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section states "the founder's leaning is a 'primary' contact who can accept proposals, approve milestones and see and pay invoices" — a distinct entitlement set from a comment-only contact, so it requires a distinct role. (BRIEF.md, Target Users & Roles.) [RESEARCH-INFORMED: no profiled competitor offers a distinct primary-versus-reviewer contact split (market-research.md, Feature Comparison Matrix), so this role is a differentiator rather than a copied convention]

**Name:** Owen

**Description:** Owen is the founder or lead at a small client company who signs off on scope and money. He is Nadia's main point of contact for anything that commits the company: accepting a proposal, approving a milestone, or paying an invoice.

**Goals:** Understand exactly what he is agreeing to and when; ask for changes to a proposal without starting an email thread [AUDIT-ADDED: 1 -- counterpart symmetry: request-changes path added to Proposal Acceptance]; approve work with confidence that it is recorded; pay invoices quickly without hunting for a payment link; keep track of where a project stands without asking Nadia for a status update.

**Pain Points:** Today he has to piece together project status from email threads and WhatsApp screenshots that other people in his company sent (BRIEF.md, Problem Statement); he has no single place to see what is due or already paid.

**Behavioral Context:** Opens the portal from an email link — a new proposal, an approval request, or an invoice — often on his phone, since he is not installing a dedicated app (BRIEF.md, Target Users & Roles). Sessions are short and triggered by a specific email, not habitual browsing.

**What This User Does NOT Need:** An account with Clientroom beyond passwordless magic-link access — no password to manage, no separate account-creation step (BRIEF.md, Target Users & Roles: "never hit an account-creation wall"); visibility into any other client's projects (BRIEF.md, Target Users & Roles: "never see another client's").

### Client Reviewer Contact

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section describes "one or two people who give feedback" alongside the person who signs and pays, and states that "not every contact may approve work or see invoices" with reviewer contacts leaning toward comment-only access. This is a distinct, lighter entitlement set from the Primary Contact's. (BRIEF.md, Target Users & Roles.)

**Name:** Priya

**Description:** Priya is a marketing lead or similar stakeholder at the client company who reviews design or content work and leaves feedback, but does not hold financial or contractual authority for the company.

**Goals:** See new deliverables as soon as they are ready; leave clear, specific feedback pinned to the actual files or to a milestone as a whole [AUDIT-ADDED: 2 -- milestone-level comments added to Deliverable Review & Feedback]; know that her comments reach Nadia without relaying them through Owen or a screenshot.

**Pain Points:** Feedback currently has to travel through WhatsApp screenshots because she has no direct, contextual way to comment on files (BRIEF.md, Problem Statement).

**Behavioral Context:** Opens the portal from an email notifying her of a new deliverable or review request, typically on her phone (BRIEF.md, The Experience: "the client's marketing lead gets an email, opens the portal on their phone"). [MODIFIED: synthesis check — citation corrected; the quoted sentence comes from BRIEF.md's The Experience section, not Target Users & Roles]

**What This User Does NOT Need:** The ability to accept proposals, see proposal content, approve milestones, or see and pay invoices — the brief's leaning reserves those to the Primary Contact (BRIEF.md, Target Users & Roles) [MODIFIED: "see proposal content" made explicit to match the Access Matrix (Proposals & Acceptance: None for Reviewers) and Proposal Acceptance (FEAT-03)]; visibility into any other client company's projects.

### Support Operator

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section states directly: "The founder, as operator, needs read-only support access to a freelancer's account, nothing more," and explicitly labels this "not a product role." It is modeled here only so the Access Matrix and per-feature Access fields can state its (deliberately minimal) entitlements, which Operator Support Access (FEAT-31) now defines. (BRIEF.md, Target Users & Roles.)

**Name:** Dana (the founder, acting in an operator capacity)

**Description:** Dana is the Clientroom founder's own support identity, used only to help a freelancer with an account issue — never a marketed or freelancer-facing role.

**Goals:** Diagnose and resolve an account problem a freelancer has reported, without needing to be granted freelancer-level control.

**Pain Points:** N/A — this is not a customer-facing persona with product pain points; its entry exists solely to bound support access.

**Behavioral Context:** Accessed only in response to a specific support request, never as routine monitoring. Every session is read-only, time-limited, announced to the freelancer by email, and listed in her activity trail. [AUDIT-ADDED: 3 -- the draft named the role but no feature defined how its access starts, stays read-only, or is made visible; now defined by Operator Support Access (FEAT-31)]

**What This User Does NOT Need:** The ability to edit, approve, send, or pay anything on a freelancer's behalf — read-only, per BRIEF.md's explicit "nothing more"; the ability to sign in as a client contact, download deliverable files, or generate data exports.

## Access Matrix

| Role / Persona | Client & Project Management | Proposals & Acceptance | Milestones & Deliverables | Invoicing & Payments | Financial Dashboard & Accounting Export | Activity & Audit Trail | Client Contact Management | Branding, Onboarding & Settings | Subscription & Account Data | Client Portal Access | Notifications & Help | Support Access | Portal Referral Mark |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Nadia (Freelancer) | Full | Full | Full | Full | Full | Full (read and share; entries are never editable by anyone) | Full | Full | Full | Full (controls who can reach each portal) | Full | View (sees every support session on her account) | View |
| Owen (Client Primary Contact) | None | Own-only (view, accept, request changes, sign) | Own-only (view, comment, approve) | Own-only (view, pay, download copies) | None | None | Own-only (invite Reviewer contacts at own company) | None | None | Own-only | Own-only (receives own emails; help tips) | None | View |
| Priya (Client Reviewer Contact) | None | None | Own-only (view, comment; no approve) | None | None | None | None | None | None | Own-only | Own-only (receives own emails; help tips) | None | View |
| Dana (Support Operator) | View | View | View (no file downloads) | View | View (dashboard only; no export generation) | View | View | View | View (plan status only; no export or deletion) | None | View (delivery warnings only) | Full (opens read-only sessions, always logged) | View |

[MODIFIED: matrix extended from 7 to 13 capability groups so that every capability group in the final feature set is covered — Activity & Audit Trail, Subscription & Account Data, Client Portal Access, Notifications & Help, Support Access, and Portal Referral Mark were added; Proposals became "Proposals & Acceptance" with request changes and signing (FEAT-03, FEAT-26); Owen's invoice access gained invoice copies (FEAT-09); Dana's cells were narrowed to match the read-only boundaries defined by Operator Support Access (FEAT-31)]

Every access level means the same thing in every feature: "Full" is create, view, change, and remove within the product's own limits (records that are evidence — acceptances, approvals, sent invoices, trail entries — are never editable by anyone); "View" is read-only; "Own-only" is limited to the contact's own client company's projects; "None" means the capability is not shown at all. A client contact who opens a link outside their scope, or a sign-in link that has expired, sees a plain explanation and a way to request a fresh link — never another company's data. [AUDIT-ADDED: 1 -- Access Matrix audit: the draft did not state what an unauthorized person experiences]
