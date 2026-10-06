# Part C — Scope & Assumptions

This part fixes the boundaries of the build: what is deliberately excluded, what the plan assumes, the constraints and non-functional expectations it must honor, and the brief's open questions that the team still owns.

## What Is Out of Scope

Every exclusion (SC-XX) with its rationale and deferral notes follows.


# Scope Boundaries

## In-Scope Summary

This product lets a solo freelancer send a proposal, set milestones and a payment schedule, share deliverables, collect client approval and feedback, and invoice and get paid straight into her own payment account — all in one branded, per-client portal. Core features (FEAT-01 through FEAT-16, plus Payment Account Connection, FEAT-32) close this loop end-to-end, including the currency/tax/time-zone, large-file, notification, and record-immutability platform capabilities it depends on. Important features (FEAT-17 through FEAT-25, plus Operator Support Access, FEAT-31, and Portal Referral Attribution, FEAT-33) round out version history, contact roles, branding, the portal-driven growth loop, onboarding, settings, bookkeeping export, subscription billing, GDPR export/deletion, refund/cancellation handling, and bounded support access. Nice-to-Have features (FEAT-26 through FEAT-30) are genuine future value — e-signature, a custom domain, search, an in-app feed, and contextual help — none of which the MVP loop depends on. [MODIFIED: summary updated for the three features added during synthesis (FEAT-31, FEAT-32, FEAT-33)]

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

## Deferral Notes

- **Global Search Across Clients & Projects** — Target phase: v1. Deferred until a freelancer's roster and history are large enough (BRIEF.md, Scale: up to 15 clients) that manual browsing genuinely slows her down.
- **Legally Binding E-Signature for Proposals** — Target phase: v1. Deferred rather than rejected, per BRIEF.md's own open framing of this question; the timestamped Accept is the launch default. Brought forward from Later because all 5 profiled competitors bundle e-signature with proposals, so freelancers switching tools will expect it soon after launch. [MODIFIED: target phase moved from Later to v1 based on e-signature presence in all 5 profiled competitors (5 sources, HIGH confidence)]
- **Custom Domain per Freelancer** — Target phase: Later. BRIEF.md itself defers the timing question ("ideally... timing is an open question"). Worth prioritizing once branding (FEAT-19) adoption shows freelancers want to go further with their own domain; white-label depth is well received in the market (SuiteDash, HIGH).
- **In-App Notification Center** — Target phase: Later. Deferred because email already satisfies BRIEF.md's stated notification channel; an in-app feed is a freelancer-side convenience layered on top once the email-driven loop is well established.
- **Contextual Help & Guidance** — Target phase: Later. Deferred as a polish layer once the core one-click flows (accept, approve, pay) are validated as self-explanatory in practice.

Deliverable Version History and Accounting Export are no longer deferred: both are now phased MVP in product-features.md. [MODIFIED: removed from Deferral Notes — version history because BRIEF.md's Scale section lists it as a day-one norm and no competitor offers it (market-research.md, Absent Features); accounting export because BRIEF.md's "v1" denotes the first release]


## Assumptions, Constraints & Expectations

Assumptions (ASMP-XX), product constraints, non-functional expectations and dependencies follow.


# Assumptions and Constraints

## Product Assumptions

### User Environment

- **ID:** ASMP-01
- **We assume freelancers work primarily from a laptop or desktop with reliable internet access.** Invalidated if: usage data shows a significant share of freelancers working mobile-only or in low-connectivity conditions when managing clients (BRIEF.md, Scale & Non-Functional Expectations states "freelancers work on laptop or desktop").

- **ID:** ASMP-02
- **We assume client contacts review and act on requests primarily from mobile browsers.** Invalidated if: usage data shows the majority of client sessions occur on desktop instead (BRIEF.md, Scale & Non-Functional Expectations: "clients mostly review on mobile browsers").

- **ID:** ASMP-03
- **We assume both freelancers and client contacts have reliable, actively-monitored email access as their primary channel.** Invalidated if: a meaningful share of target users do not check email regularly, or notifications are consistently spam-filtered, since email is the product's sole client-facing communication channel (BRIEF.md, Ecosystem & Integrations). [RESEARCH-INFORMED: client emails landing in spam are reported for HoneyBook and SuiteDash (MEDIUM), so this assumption is watched through the notification-delivery metric in success-metrics.md]

- **ID:** ASMP-04
- **We assume freelancers in the target markets can open, or already hold, an account with an established payment processor that accepts card and bank-transfer payments in their country.** Invalidated if: a significant share of sign-ups cannot connect a payment account (Payment Account Connection, FEAT-32) because no supported processor serves their country or business type, leaving them to record every payment manually. [AUDIT-ADDED: 1 -- value-flow walk: the worldwide-from-day-one goal in BRIEF.md depends on this]

### User Behavior

- **ID:** ASMP-05
- **We assume freelancers will define milestones and a payment schedule before starting work, rather than invoicing ad hoc after the fact.** Invalidated if: usage shows most freelancers skip milestone setup entirely and rely only on manual, ad-hoc invoicing.

- **ID:** ASMP-06
- **We assume client Primary Contacts will act on approval and payment requests within days, not weeks, once the flow is this frictionless.** Invalidated if: median approval and payment turnaround remains multi-week despite automated reminders, suggesting the friction the brief describes was not primarily tooling-related.

- **ID:** ASMP-07
- **We assume reviewer contacts will adopt pinned, in-portal comments in place of outside channels like WhatsApp once the option exists.** Invalidated if: clients continue relying on outside channels for feedback alongside or instead of the portal.

- **ID:** ASMP-08
- **We assume freelancers prefer fixed, sensible behavior (automatic invoice on approval, day-3 and day-10 reminders) over configuring their own workflows.** Invalidated if: a significant share of freelancers ask for custom workflow or reminder configuration, or churn citing lack of customization. [RESEARCH-INFORMED: the category's dominant complaint is a 15–25+ hour configuration burden (Dubsado, SuiteDash; independent reviews, HIGH), while HoneyBook users also ask for more customization (MEDIUM) — this assumption bets on the first finding]

### Product Context

- **ID:** ASMP-09
- **We assume this product stands alone rather than as an add-on to an existing project-management or invoicing suite.** Invalidated if: target freelancers strongly prefer a plug-in to a tool they already use over a standalone product (BRIEF.md, Ecosystem & Integrations: "Otherwise the product stands alone (confirmed)").

- **ID:** ASMP-10
- **We assume solo freelancers, not agencies or teams, are the addressable market for v1.** Invalidated if: significant, sustained demand emerges from small agencies wanting multi-seat, team-based access (BRIEF.md, Target Users & Roles: "Agencies with several team members are out of scope").

- **ID:** ASMP-11
- **We assume a recorded, timestamped Accept is enough evidence for most freelancers' scope disputes, without a legally binding e-signature.** Invalidated if: freelancers report clients or disputes where a plain Accept was not accepted as agreement, or ask for e-signature before the v1 release of Legally Binding E-Signature for Proposals (FEAT-26). [RESEARCH-INFORMED: all 5 profiled competitors bundle e-signature with proposals, which is why FEAT-26 moved to v1; BRIEF.md's Open Questions frame the timestamped Accept as the presumptive default]

- **ID:** ASMP-12
- **We assume a permanent free tier for one or two active clients will convert enough freelancers to paid plans and feed the portal-driven growth loop.** Invalidated if: free-to-paid conversion (success-metrics.md) stays well below target while free accounts consume storage and support time. [CHALLENGED: every profiled competitor uses a 7–30 day trial and none offers an ongoing free tier (vendor pricing pages, 5 sources, HIGH confidence) -- original retained per SYN-04 protection (user-stated pricing model in BRIEF.md, Business Context)]

## Product Constraints

- **ID:** ASMP-13
- **Solo-freelancer-only platform** — A deliberate v1 scoping choice, not a technical limitation: no team seats, internal role hierarchy, or agency-style resourcing exists in this product (BRIEF.md, Target Users & Roles).

- **ID:** ASMP-14
- **The platform never holds or moves client funds** — A deliberate trust and regulatory-exposure constraint: all payments go directly into the freelancer's own processor account, connected by the freelancer herself (FEAT-32), even though many competing products in this space do hold funds (BRIEF.md, Constraints). [RESEARCH-INFORMED: Bonsai users report payouts held for up to 10 business days (MEDIUM), the failure mode this constraint rules out]

- **ID:** ASMP-15
- **Records are append-only and immutable once created** — A deliberate constraint protecting the product's evidentiary value: accepted proposals, approvals, and sent invoices can never be silently edited, even where in-place correction would be more convenient (BRIEF.md, Constraints).

- **ID:** ASMP-16
- **Web-only, no native apps for v1** — A deliberate scope constraint keeping the build achievable for a solo founder within roughly three months, rather than a statement that native apps have no value (BRIEF.md, Constraints; Scale & Non-Functional Expectations).

- **ID:** ASMP-17
- **No platform fee or cut of payments** — A deliberate pricing-model constraint distinguishing this product from competitors that take a percentage of processed payments; revenue comes solely from the freelancer's subscription (BRIEF.md, Business Context). [RESEARCH-INFORMED: HoneyBook charges 2.7%+10¢ per card payment and Bonsai about 3% on top of their subscriptions, a sustained complaint in G2 and Trustpilot reviews (HIGH)]

- **ID:** ASMP-18
- **Support access is read-only and always visible to the freelancer** — A deliberate trust constraint: the operator can only look, never change, and every support session is announced to the freelancer and kept in her activity trail (FEAT-31), per BRIEF.md's "read-only support access to a freelancer's account, nothing more." [AUDIT-ADDED: 3 -- the Support Operator role's boundary made a stated constraint]

- **ID:** ASMP-19
- **English-only interface at launch, with locale-aware money, dates, and time zones** — A deliberate launch constraint from BRIEF.md ("English only at launch") that still honors its worldwide requirement that currencies, tax lines, and time zones are never hard-coded (FEAT-15). [AUDIT-ADDED: 4 -- internationalization concern]

- **ID:** ASMP-20
- **Evidence outlives a contact's erasure request, but only as far as needed** — When a client contact asks to be erased, their access ends and their contact details are removed, but acceptances and approvals they gave stay on the record under their name, because those records are the freelancer's evidence of what was agreed (BRIEF.md, Constraints: record immutability and GDPR). This is a deliberate balance between the two brief constraints; the legal basis for keeping evidence records is to be confirmed with qualified privacy advice before launch. [AUDIT-ADDED: 4 -- compliance: record immutability and the right to erasure pull in opposite directions, so the boundary is stated rather than left implicit]

## Non-Functional Expectations

- **ID:** ASMP-21
- **Responsiveness: client-facing pages (deliverable review, approval, invoice payment) become interactive within roughly 2 seconds on a typical mobile connection; dashboard totals appear within roughly 1–2 seconds.** — Basis: BRIEF.md, Scale & Non-Functional Expectations ("the client side must be excellent on mobile"). [RESEARCH-INFORMED: slow page loads are the single most cited complaint in negative SuiteDash reviews and are reported for HoneyBook (HIGH)]

- **ID:** ASMP-22
- **Data volume and growth: a few thousand freelancers in year one, each with 3–15 active clients and a handful of contacts per client; deliverables typically tens of MB and sometimes over 1 GB, retained with version history for the life of the account.** — Basis: BRIEF.md, Scale & Non-Functional Expectations. [MODIFIED: synthesis check — "indefinitely" replaced by "for the life of the account" so it agrees with account deletion (FEAT-24) and scope-boundaries.md]

- **ID:** ASMP-23
- **Privacy posture: strict data isolation between clients (a client never sees another client's anything), freelancers can export and delete their own data on request, and any operator access is read-only and visible to the freelancer.** — Basis: BRIEF.md, Privacy and Constraints; Target Users & Roles (operator support access).

- **ID:** ASMP-24
- **Compliance: personal data of freelancers and client contacts worldwide is treated as GDPR-class personal data; no card or payment data is ever captured or stored by the product itself, since that handling belongs entirely to the payment-processing capability; invoices carry the content commonly required of a valid invoice (sequential number, both parties' business details, issue and due dates, tax line).** — Basis: BRIEF.md, Privacy and Constraints; domain reasoning from the cross-cutting decomposition checklist (Compliance). [AUDIT-ADDED: 4 -- invoice-content compliance added]

- **ID:** ASMP-25
- **Correctness of financial and evidentiary records: accepted proposals, approvals, and sent invoices are timestamped at creation and never silently altered afterward, regardless of account or usage scale.** — Basis: BRIEF.md, Constraints ("payments and records must be correct").

- **ID:** ASMP-26
- **Availability and delivery: no specific uptime target is committed; the product is expected to be reliably available during ordinary business use, and emails that fail to deliver are surfaced to the freelancer within minutes rather than lost.** — Basis: BRIEF.md, Scale & Non-Functional Expectations ("no specific uptime number was stated"). [RESEARCH-INFORMED: messages failing to send or landing in spam are reported across four profiled products (HIGH), so delivery visibility is part of the reliability expectation]

- **ID:** ASMP-27
- **Accessibility and degraded states: client-facing screens are readable on a phone without zooming, usable with a screen reader and keyboard, never rely on colour alone, and keep brand colours legible; every screen shows real progress while loading, keeps typed input on errors, and says plainly when an action needs a connection — actions that create records (accept, approve, pay) never pretend to succeed offline.** — Basis: BRIEF.md, Scale & Non-Functional Expectations (clients mostly on mobile; client side must be excellent on mobile) and Constraints (records must be correct); decomposition checklist Commonly Forgotten Areas (accessibility baseline, loading states, offline posture). [AUDIT-ADDED: 4 -- product-level accessibility, loading, and offline conventions decided, matching each feature's States field]

## Dependencies

- **ID:** ASMP-28
- **Payment-processing capability** — The product requires the ability to accept card and bank-transfer payments directly into each freelancer's own account, to let each freelancer connect her own account, and to report payment status, pending bank transfers, and reversals back to the product. Without it, invoices cannot be paid in-portal at all, and the "pay it by card on the spot" experience described in BRIEF.md's Experience narrative cannot exist. [MODIFIED: extended with per-freelancer account connection and status reporting, based on the value-flow walk that added Payment Account Connection (FEAT-32) and reversal handling (FEAT-25)]

- **ID:** ASMP-29
- **Transactional email delivery capability** — The product requires reliable email delivery for every notification: proposals, deliverable-ready alerts, approval requests, invoices, reminders, and support-session notices, with delivery and bounce status reported back. Without it, no client-facing communication reaches its recipient, since BRIEF.md states plainly that "clients will not install an app."

- **ID:** ASMP-30
- **File storage and delivery capability for large files** — The product requires the ability to store and reliably deliver deliverables from a few MB up to 1 GB+, with version history, within a modest budget. Without it, the deliverable-sharing loop central to the product cannot function at the scale BRIEF.md describes.

- **ID:** ASMP-31
- **Subscription-billing capability for the freelancer's own plan** — The product requires the ability to charge freelancers a recurring subscription once they exceed the free tier. Without it, the business model described in BRIEF.md's Business Context cannot operate, independent of and separate from the client-side payment-processing capability (ASMP-28).

- **ID:** ASMP-32
- **Domain-verification capability (Later phase)** — Custom Domain per Freelancer (FEAT-27) requires the ability to verify that a freelancer controls a domain and serve her portal securely at it. Without it, only the shared default portal address is available; nothing in the MVP depends on it. [AUDIT-ADDED: 4 -- external-service integrations concern: the dependency behind a roadmap feature stated in advance]


## Open Items for the Team

These are the brief's open questions — unresolved by design; the team owns them.

- **Client contact roles:** who can accept proposals and approve work, who can see and pay invoices, and who can invite colleagues? The leaning is "primary" versus comment-only "reviewer" contacts, but it is not decided.
- **Proposal acceptance:** do proposals need legally binding e-signatures, or is a recorded, timestamped "Accept" enough?
- **Time tracking:** is it in scope, or does it belong in another tool?
- **Tax handling depth:** should invoices only show a tax line, or calculate tax per country or region (VAT, GST, US sales tax)?
- **Custom domain per freelancer:** in v1, or later?
- **Refunds and disputes:** refunds are assumed to be issued by the freelancer through the processor from their own account, but the refund, chargeback and cancelled-project flow was not defined.
- **Pricing:** the exact subscription price points and the free-tier limit (one or two active clients) are not set.
- **Large-file cost:** how do we store and deliver files of tens of MB up to 1 GB+ with version history within roughly $100/month of infrastructure?

