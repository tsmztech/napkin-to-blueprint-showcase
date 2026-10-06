---
document_type: assumptions-constraints
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed (1 fix applied)
---

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
