---
document_type: assumptions-constraints
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# Assumptions and Constraints

## Product Assumptions

### User Environment

- **ID:** ASMP-01
- **We assume freelancers work primarily from a laptop or desktop with reliable internet access.** Invalidated if: usage data shows a significant share of freelancers working mobile-only or in low-connectivity conditions when managing clients (BRIEF.md, Scale & Non-Functional Expectations states "freelancers work on laptop or desktop").

- **ID:** ASMP-02
- **We assume client contacts review and act on requests primarily from mobile browsers.** Invalidated if: usage data shows the majority of client sessions occur on desktop instead (BRIEF.md, Scale & Non-Functional Expectations: "clients mostly review on mobile browsers").

- **ID:** ASMP-03
- **We assume both freelancers and client contacts have reliable, actively-monitored email access as their primary channel.** Invalidated if: a meaningful share of target users do not check email regularly, or notifications are consistently spam-filtered, since email is the product's sole client-facing communication channel (BRIEF.md, Ecosystem & Integrations).

### User Behavior

- **ID:** ASMP-04
- **We assume freelancers will define milestones and a payment schedule before starting work, rather than invoicing ad hoc after the fact.** Invalidated if: usage shows most freelancers skip milestone setup entirely and rely only on manual, ad-hoc invoicing.

- **ID:** ASMP-05
- **We assume client Primary Contacts will act on approval and payment requests within days, not weeks, once the flow is this frictionless.** Invalidated if: median approval and payment turnaround remains multi-week despite automated reminders, suggesting the friction the brief describes was not primarily tooling-related.

- **ID:** ASMP-06
- **We assume reviewer contacts will adopt pinned, in-portal comments in place of outside channels like WhatsApp once the option exists.** Invalidated if: clients continue relying on outside channels for feedback alongside or instead of the portal.

### Product Context

- **ID:** ASMP-07
- **We assume this product stands alone rather than as an add-on to an existing project-management or invoicing suite.** Invalidated if: target freelancers strongly prefer a plug-in to a tool they already use over a standalone product (BRIEF.md, Ecosystem & Integrations: "Otherwise the product stands alone (confirmed)").

- **ID:** ASMP-08
- **We assume solo freelancers, not agencies or teams, are the addressable market for v1.** Invalidated if: significant, sustained demand emerges from small agencies wanting multi-seat, team-based access (BRIEF.md, Target Users & Roles: "Agencies with several team members are out of scope").

## Product Constraints

- **ID:** ASMP-09
- **Solo-freelancer-only platform** — A deliberate v1 scoping choice, not a technical limitation: no team seats, internal role hierarchy, or agency-style resourcing exists in this product (BRIEF.md, Target Users & Roles).

- **ID:** ASMP-10
- **The platform never holds or moves client funds** — A deliberate trust and regulatory-exposure constraint: all payments go directly into the freelancer's own processor account, even though many competing products in this space do hold funds (BRIEF.md, Constraints).

- **ID:** ASMP-11
- **Records are append-only and immutable once created** — A deliberate constraint protecting the product's evidentiary value: accepted proposals, approvals, and sent invoices can never be silently edited, even where in-place correction would be more convenient (BRIEF.md, Constraints).

- **ID:** ASMP-12
- **Web-only, no native apps for v1** — A deliberate scope constraint keeping the build achievable for a solo founder within roughly three months, rather than a statement that native apps have no value (BRIEF.md, Constraints; Scale & Non-Functional Expectations).

- **ID:** ASMP-13
- **No platform fee or cut of payments** — A deliberate pricing-model constraint distinguishing this product from competitors that take a percentage of processed payments; revenue comes solely from the freelancer's subscription (BRIEF.md, Business Context).

## Non-Functional Expectations

- **ID:** ASMP-14
- **Responsiveness: client-facing pages (deliverable review, approval, invoice payment) become interactive within roughly 2 seconds on a typical mobile connection; dashboard totals appear within roughly 1–2 seconds.** — Basis: BRIEF.md, Scale & Non-Functional Expectations ("the client side must be excellent on mobile").

- **ID:** ASMP-15
- **Data volume and growth: a few thousand freelancers in year one, each with 3–15 active clients and a handful of contacts per client; deliverables typically tens of MB and sometimes over 1 GB, retained with version history indefinitely.** — Basis: BRIEF.md, Scale & Non-Functional Expectations.

- **ID:** ASMP-16
- **Privacy posture: strict data isolation between clients (a client never sees another client's anything), and freelancers can export and delete their own data on request.** — Basis: BRIEF.md, Privacy and Constraints.

- **ID:** ASMP-17
- **Compliance: personal data of freelancers and client contacts worldwide is treated as GDPR-class personal data; no card or payment data is ever captured or stored by the product itself, since that handling belongs entirely to the payment-processing capability.** — Basis: BRIEF.md, Privacy and Constraints; domain reasoning from the cross-cutting decomposition checklist (Compliance).

- **ID:** ASMP-18
- **Correctness of financial and evidentiary records: accepted proposals, approvals, and sent invoices are timestamped at creation and never silently altered afterward, regardless of account or usage scale.** — Basis: BRIEF.md, Constraints ("payments and records must be correct").

- **ID:** ASMP-19
- **Availability: no specific uptime target is committed; the product is expected to be reliably available during ordinary business use, but no formal service-level commitment applies at this stage.** — Basis: BRIEF.md, Scale & Non-Functional Expectations ("no specific uptime number was stated").

## Dependencies

- **ID:** ASMP-20
- **Payment-processing capability** — The product requires the ability to accept card and bank-transfer payments directly into each freelancer's own account. Without it, invoices cannot be paid in-portal at all, and the "pay it by card on the spot" experience described in BRIEF.md's Experience narrative cannot exist.

- **ID:** ASMP-21
- **Transactional email delivery capability** — The product requires reliable email delivery for every notification: proposals, deliverable-ready alerts, approval requests, invoices, and reminders. Without it, no client-facing communication reaches its recipient, since BRIEF.md states plainly that "clients will not install an app."

- **ID:** ASMP-22
- **File storage and delivery capability for large files** — The product requires the ability to store and reliably deliver deliverables from a few MB up to 1 GB+, with version history, within a modest budget. Without it, the deliverable-sharing loop central to the product cannot function at the scale BRIEF.md describes.

- **ID:** ASMP-23
- **Subscription-billing capability for the freelancer's own plan** — The product requires the ability to charge freelancers a recurring subscription once they exceed the free tier. Without it, the business model described in BRIEF.md's Business Context cannot operate, independent of and separate from the client-side payment-processing capability (ASMP-20).
