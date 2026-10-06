---
document_type: scope-boundaries
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# Scope Boundaries

## In-Scope Summary

This product lets a solo freelancer send a proposal, set milestones and a payment schedule, share deliverables, collect client approval and feedback, and invoice and get paid — all in one branded, per-client portal. Core features (FEAT-01 through FEAT-16) close this loop end-to-end, including the currency/tax, large-file, notification, and record-immutability platform capabilities it depends on. Important features (FEAT-17 through FEAT-25) round out contact roles, branding, onboarding, settings, bookkeeping export, subscription billing, GDPR export/deletion, and refund/cancellation handling. Nice-to-Have features (FEAT-26 through FEAT-30) are genuine future value — e-signature, a custom domain, search, an in-app feed, and contextual help — none of which the MVP loop depends on.

## Explicit Exclusions

### User Scope Exclusions

- **ID:** SC-01
- **Agency or team-of-many accounts** — BRIEF.md's Target Users & Roles states plainly: "Solo freelancers only for v1. Agencies with several team members are out of scope." There is no internal-staff seat model in this product.

- **ID:** SC-02
- **Client-side roles beyond Primary and Reviewer** — The persona set in user-persona.md establishes exactly these two client-contact roles, per the founder's leaning described in BRIEF.md's Target Users & Roles; no further tiers (e.g., a client-side "admin" distinct from Primary) are modeled without a brief signal for one.

- **ID:** SC-03
- **Public or anonymous portal access** — BRIEF.md's Constraints require strict isolation between clients; only contacts the freelancer (or a Primary contact) has explicitly added can ever reach a client's portal view.

### Feature Scope Exclusions

- **ID:** SC-04
- **Time tracking** — BRIEF.md's Open Questions leaves this explicitly unresolved ("is it in scope, or does it belong in another tool?"). The product's defined value is proposals, milestones, deliverables, and invoices — not time-based billing — and the brief's own critique of tool sprawl argues against adding a capability the brief itself is unsure belongs here.

- **ID:** SC-05
- **Native mobile apps** — BRIEF.md, Scale & Non-Functional Expectations states "web app... No native apps." The client experience must be excellent on mobile browsers instead.

- **ID:** SC-06
- **Live, two-way accounting sync** — BRIEF.md, Ecosystem & Integrations: "v1 provides an export file (CSV or a QuickBooks/Xero-compatible format). No live sync." A generated export file (FEAT-22) is the full extent of the accounting integration.

- **ID:** SC-07
- **Copying or hosting files from Figma, Google Drive, or Dropbox** — BRIEF.md, Ecosystem & Integrations: these tools are "accepted as deliverables by link, not copied into Clientroom, for v1." Deliverable Upload & Sharing (FEAT-06) references linked assets; it does not mirror their content.

- **ID:** SC-08
- **General task/project-management tooling (boards, resourcing, team scheduling)** — BRIEF.md's Problem Statement explicitly positions this product against "agency project-management suites [that] are heavy, priced per seat, and built for teams of 20"; adding general PM machinery would recreate the exact tool this product is meant to replace.

- **ID:** SC-09
- **The platform holding or moving client funds** — BRIEF.md, Constraints: "the platform never holds or moves funds and never stores card data." Every payment goes directly into the freelancer's own processor account.

### Scale Expectations

- **ID:** SC-10
- **Freelancer and client roster scale** — Expected: a few thousand freelancers in year one, each with 3–15 active clients and a handful of contacts per client (BRIEF.md, Scale & Non-Functional Expectations); the product is designed to stay responsive at this scale from MVP onward, not phased in.

- **ID:** SC-11
- **Large-file deliverables within a modest infrastructure budget** — Expected: files typically tens of MB and sometimes over 1 GB for video, with version history per deliverable, kept within a roughly $100/month infrastructure budget until revenue changes that math (BRIEF.md, Scale & Non-Functional Expectations; Constraints). This is honored from MVP via Large File Handling & Storage (FEAT-16), not deferred, since the brief's target users produce large files from day one.

- **ID:** SC-12
- **Worldwide currency, tax-line, and time-zone support from day one** — Expected: the product serves freelancers and clients worldwide immediately, per BRIEF.md's Scale & Non-Functional Expectations ("Currencies, tax on invoices... and time zones must not be hard-coded"); this is an MVP requirement (Currency & Tax Handling, FEAT-15), not a later phase-in.

- **ID:** SC-13
- **Indefinite retention of evidentiary records** — Expected: accepted proposals, approvals, and sent invoices are retained permanently as the freelancer's evidence in scope disputes (BRIEF.md, Constraints: record immutability), with no automatic purge of this history at any scale.

- **ID:** SC-14
- **No committed uptime target** — Genuinely out of vision for this document: BRIEF.md, Scale & Non-Functional Expectations states plainly "no specific uptime number was stated," so this is left as an open operational decision rather than a product-scope commitment.

## Deferral Notes

- **Deliverable Version History** — Target phase: v1. Deferred rather than rejected. The MVP's single-round upload-and-approve loop must be proven first; version confusion becomes visible once clients start requesting multiple revision rounds, which is when this earns its place.
- **Accounting Export** — Target phase: v1. Deferred to match BRIEF.md's own stated v1 scope for this integration; the freelancer's first invoices can be reconciled manually until export volume justifies it.
- **Global Search Across Clients & Projects** — Target phase: v1. Deferred until a freelancer's roster and history are large enough (BRIEF.md, Scale: up to 15 clients) that manual browsing genuinely slows her down.
- **Legally Binding E-Signature for Proposals** — Target phase: Later. Deferred rather than rejected, per BRIEF.md's own open framing of this question. Worth revisiting if freelancers report client-side resistance to a plain timestamped Accept as contractually sufficient.
- **Custom Domain per Freelancer** — Target phase: Later. BRIEF.md itself defers the timing question ("ideally... timing is an open question"). Worth prioritizing once branding (FEAT-19) adoption shows freelancers want to go further with their own domain.
- **In-App Notification Center** — Target phase: Later. Deferred because email already satisfies BRIEF.md's stated notification channel; an in-app feed is a freelancer-side convenience layered on top once the email-driven loop is well established.
- **Contextual Help & Guidance** — Target phase: Later. Deferred as a polish layer once the core one-click flows (accept, approve, pay) are validated as self-explanatory in practice.
