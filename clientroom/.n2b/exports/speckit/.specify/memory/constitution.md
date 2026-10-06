# Clientroom Constitution

## Core Principles

### I. Blueprint fidelity

The specifications under `specs/` are rendered from the Clientroom blueprint package
(`docs/blueprint/` — the canonical source of truth). Specs are the contract: build what
they say. FEAT, SPEC, and acceptance-criterion IDs are never edited, renumbered, or
dropped; when a spec and an implementation convenience conflict, the spec wins. The
blueprint copy is read-only — a defect there is reported to a human, never patched here.

### II. The recommended architecture is binding

`docs/blueprint/architecture/technical-architecture.md` records a RECOMMENDED choice for
every decision area, plus documented alternatives with `Choose instead when` conditions.
Build the recommendation. The alternatives are informational — they are the humans' to
weigh, not the agent's to choose. Each feature's `research.md` restates the applicable
decisions as resolved facts.

### III. Scope boundaries (DO-NOT-BUILD)

The exclusions below are carried verbatim from the blueprint
(`docs/blueprint/features/scope-boundaries.md`). Never implement any of them — even when
a spec seems adjacent or the capability seems easy to add. (ID-space note: these SC-XX
IDs are n2b scope-boundary IDs; the SC-001-style entries inside each spec.md's
Measurable Outcomes are Spec Kit's own per-feature numbering — two different ID spaces.)

## Explicit Exclusions

### User Scope Exclusions

- **ID:** SC-01
- **Agency or team-of-many accounts** — BRIEF.md's Target Users & Roles states plainly: "Solo freelancers only for v1. Agencies with several team members are out of scope." There is no internal-staff seat model in this product, including the scoped bookkeeper or contractor access that Dubsado users ask for. [RESEARCH-INFORMED: freelancer-side scoped-permission demand is documented (Dubsado, MEDIUM) but belongs to team accounts, which the brief excludes]

- **ID:** SC-02
- **Client-side roles beyond Primary and Reviewer** — The persona set in user-persona.md establishes exactly these two client-contact roles, per the founder's leaning described in BRIEF.md's Target Users & Roles; no further tiers (e.g., a client-side "admin" distinct from Primary) are modeled without a brief signal for one. [RESEARCH-INFORMED: no profiled competitor offers even the two-role split (market-research.md, Feature Comparison Matrix), so there is no market evidence pulling toward additional tiers]

- **ID:** SC-03
- **Public or anonymous portal access** — BRIEF.md's Constraints require strict isolation between clients; only contacts the freelancer (or a Primary contact) has explicitly added can ever reach a client's portal view. The referral mark (FEAT-33) leads only to a public product page and never exposes portal content.

- **ID:** SC-04
- **Operator changes to a freelancer's account, or signing in as a client contact** — BRIEF.md, Target Users & Roles: the operator "needs read-only support access to a freelancer's account, nothing more." Support sessions (FEAT-31) are read-only and logged; the operator never edits, sends, approves, pays, downloads deliverable files, or acts as a client contact. [AUDIT-EXCLUDED: 3 -- the Support Operator role's boundary made explicit when its feature was defined]

### Feature Scope Exclusions

- **ID:** SC-05
- **Time tracking** — BRIEF.md's Open Questions leaves this explicitly unresolved ("is it in scope, or does it belong in another tool?"). The product's defined value is proposals, milestones, deliverables, and invoices — not time-based billing — and the brief's own critique of tool sprawl argues against adding a capability the brief itself is unsure belongs here. [RESEARCH-INFORMED: time tracking appears in only 2 of 5 profiled competitors (Bonsai, Moxie), so leaving it out is not a table-stakes gap]

- **ID:** SC-06
- **Native mobile apps** — BRIEF.md, Scale & Non-Functional Expectations states "web app... No native apps." The client experience must be excellent on mobile browsers instead.

- **ID:** SC-07
- **Live, two-way accounting sync** — BRIEF.md, Ecosystem & Integrations: "v1 provides an export file (CSV or a QuickBooks/Xero-compatible format). No live sync." A generated export file (FEAT-22) is the full extent of the accounting integration.

- **ID:** SC-08
- **Copying or hosting files from Figma, Google Drive, or Dropbox** — BRIEF.md, Ecosystem & Integrations: these tools are "accepted as deliverables by link, not copied into Clientroom, for v1." Deliverable Upload & Sharing (FEAT-06) references linked assets; it does not mirror their content.

- **ID:** SC-09
- **General task/project-management tooling (boards, resourcing, team scheduling)** — BRIEF.md's Problem Statement explicitly positions this product against "agency project-management suites [that] are heavy, priced per seat, and built for teams of 20"; adding general PM machinery would recreate the exact tool this product is meant to replace.

- **ID:** SC-10
- **The platform holding or moving client funds** — BRIEF.md, Constraints: "the platform never holds or moves funds and never stores card data." Every payment goes directly into the freelancer's own processor account, connected through Payment Account Connection (FEAT-32).

- **ID:** SC-11
- **A configurable workflow, form, or automation builder** — Customizable workflows are central to Dubsado, SuiteDash, and HoneyBook, and they are exactly what drives the category's 15–25+ hour setup burden (independent reviews, HIGH); this product ships fixed, sensible behavior (auto-invoice on approval, day-3 and day-10 reminders) instead, in line with BRIEF.md's "the freelancer stops chasing" promise. [AUDIT-EXCLUDED: 2 -- competitor workflow builders excluded; configuration overhead is the documented pain this product avoids]

- **ID:** SC-12
- **Bundled business-management extras: lead-capture forms, client questionnaires, appointment scheduling, sales pipeline, expense tracking, tax-preparation tools, and course hosting** — Each appears in one or more profiled competitors (HoneyBook, Dubsado, Bonsai, Moxie, SuiteDash), but BRIEF.md's product starts at the proposal and ends at the paid invoice; these extras are the "five tools" sprawl and heavy-suite clutter the brief's Problem Statement argues against. [AUDIT-EXCLUDED: 2 -- competitor extras outside the proposal-to-paid vision in BRIEF.md]

- **ID:** SC-13
- **A library of legal contract templates** — Lawyer-vetted templates are praised in Bonsai reviews (MEDIUM), but providing legal documents across worldwide jurisdictions is outside a solo founder's capacity and outside BRIEF.md's scope; freelancers write their own scope and can reuse earlier proposals (FEAT-02). [AUDIT-EXCLUDED: 2 -- legal-template differentiator excluded on vision and capacity grounds]

- **ID:** SC-14
- **Recurring or retainer billing on a fixed calendar schedule** — Recurring billing appears in SuiteDash, but BRIEF.md defines billing as deposit, per milestone, on completion, or a mix; an occasional retainer charge can be issued as an ad-hoc invoice (FEAT-09). [AUDIT-EXCLUDED: 2 -- recurring billing differentiator excluded; not part of the brief's project-based payment schedules]

- **ID:** SC-15
- **A general-purpose chat or messaging inbox** — Several competitor portals include messaging (HoneyBook, Moxie), but BRIEF.md's goal is feedback that is pinned and recorded, not another chat channel; comments on deliverables and milestones (FEAT-07) and request-changes notes on proposals (FEAT-03) cover the need. [AUDIT-EXCLUDED: 2 -- open messaging excluded in favor of contextual comments]

- **ID:** SC-16
- **Automatic tax calculation per country or region** — BRIEF.md's Open Questions leaves tax depth undecided; invoices carry a freelancer-configured tax label and rate (FEAT-15). Tax tooling appears in only 1 of 5 competitors and is US-specific there (Bonsai), and maintaining worldwide tax rules is beyond a solo founder's three-month build. [AUDIT-EXCLUDED: 2 -- automatic tax calculation excluded; configured tax line retained]

- **ID:** SC-17
- **Partial payments or instalments on a single invoice** — An invoice is either unpaid or paid in full (FEAT-10); instalments are expressed as separate milestone or deposit invoices through the payment schedule (FEAT-04), which keeps every amount traceable to what was agreed. [AUDIT-EXCLUDED: 1 -- value-flow walk: partial payment deliberately routed through the schedule instead]

- **ID:** SC-18
- **Issuing refunds or fighting chargebacks inside Clientroom** — BRIEF.md: the platform never holds or moves funds, and refunds are assumed to be issued by the freelancer through her own processor account. Clientroom records refunds, partial refunds, and reported reversals truthfully (FEAT-25), but the money movement and the dispute response happen in the freelancer's processor account. [AUDIT-EXCLUDED: 1 -- value-flow walk: ownership of the refund and chargeback segment stated explicitly]

- **ID:** SC-19
- **Bulk import of clients, projects, or invoice history from other tools** — A freelancer has 3–15 active clients (BRIEF.md, Scale), so adding them by hand takes minutes; importing historical proposals and invoices would create records that were never accepted or sent through Clientroom and so cannot carry its evidence guarantee. Import failures are also a reported pain in Bonsai and Moxie (MEDIUM, LOW). [AUDIT-EXCLUDED: 4 -- data import concern excluded with rationale]

- **ID:** SC-20
- **Interfaces in languages other than English** — BRIEF.md, Scale & Non-Functional Expectations: "English only at launch." Currencies, tax lines, time zones, and date formats still adapt to each user (FEAT-15). [AUDIT-EXCLUDED: 4 -- internationalization concern: language excluded, locale formats included]

### Scale Expectations

- **ID:** SC-21
- **Freelancer and client roster scale** — Expected: a few thousand freelancers in year one, each with 3–15 active clients and a handful of contacts per client (BRIEF.md, Scale & Non-Functional Expectations); the product is designed to stay responsive at this scale from MVP onward, not phased in.

- **ID:** SC-22
- **Large-file deliverables within a modest infrastructure budget** — Expected: files typically tens of MB and sometimes over 1 GB for video, with version history per deliverable, kept within a roughly $100/month infrastructure budget until revenue changes that math (BRIEF.md, Scale & Non-Functional Expectations; Constraints). This is honored from MVP via Large File Handling & Storage (FEAT-16) and Deliverable Version History (FEAT-17), not deferred, since the brief's target users produce large files from day one. [MODIFIED: synthesis check — Deliverable Version History (FEAT-17) named here now that it is phased MVP; the draft committed to version history from MVP while phasing the feature at v1]

- **ID:** SC-23
- **Worldwide currency, tax-line, and time-zone support from day one** — Expected: the product serves freelancers and clients worldwide immediately, per BRIEF.md's Scale & Non-Functional Expectations ("Currencies, tax on invoices... and time zones must not be hard-coded"); this is an MVP requirement (Currency & Tax Handling, FEAT-15), not a later phase-in.

- **ID:** SC-24
- **Long-term retention of evidentiary records** — Expected: accepted proposals, approvals, and sent invoices are kept permanently for as long as the freelancer's account exists, as her evidence in scope disputes (BRIEF.md, Constraints: record immutability), with no automatic purge of this history at any scale. When she deletes her account (FEAT-24), they are removed except where a legal retention period for financial records applies. [MODIFIED: synthesis check — "retained permanently" qualified so it no longer contradicts account deletion (FEAT-24) and BRIEF.md's requirement that freelancers can delete their data]

- **ID:** SC-25
- **No committed uptime target** — Genuinely out of vision for this document: BRIEF.md, Scale & Non-Functional Expectations states plainly "no specific uptime number was stated," so this is left as an operational decision rather than a product-scope commitment.


### IV. Design posture

This package ships design-agnostic: no design system is part of the blueprint, and the
builder owns visual design. Honor any stated design preferences recorded in the brief's
Constraints section (`docs/blueprint/BRIEF.md`).

### V. Definition of done

A feature is done when its spec.md acceptance scenarios all pass end-to-end — every
acceptance criterion, by ID, exercised the way a user would. Not merely compiling, not
unit tests alone.

## Governance

This constitution supersedes ad-hoc practices for all work in this workspace. Amendments
require the humans who own the project; the blueprint itself is regenerated upstream,
never amended here. Every `/speckit.plan` and `/speckit.implement` run is expected to
comply with Principles I–V; complexity that violates them must be justified to a human
before it lands.

Version: 1.0.0 | Ratified: 2026-10-02 | Last Amended: 2026-10-02
