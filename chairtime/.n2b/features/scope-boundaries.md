---
document_type: scope-boundaries
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-21
synthesis_check: passed (1 fix applied)
---

# Scope Boundaries

## In-Scope Summary

This product lets one solo beauty or wellness professional (Mara) run deposit-protected bookings entirely through a shared link, with clients (Taylor) booking, paying, and managing their own appointments without an account. Core features (17) close that loop end-to-end — public booking, live availability, identity capture, deposit payment, confirmation and reminders, self-service reschedule/cancellation (client- and Pro-initiated), the daily dashboard, no-show/forfeiture handling, business configuration, manual blocking, client records, calendar sync, onboarding, account, payment and messaging capabilities, and subscription billing. Important features (4) close the brief's stated dispute-record gap and the account/support lifecycle. Nice-to-Have features (8) answer the brief's own open questions (waitlist, tipping, in-app balance payment, recurring bookings) plus domain-standard conveniences (import, export) — all deferred, none dropped. Market research confirmed every common competitor feature (online booking, card-on-file deposits, automated reminders, integrated payment processing, client management) is present in this scope, and confirmed that the category's marketplace, per-new-client-fee, and multi-staff differentiators are deliberately excluded below, not accidentally omitted.

## Explicit Exclusions

### User Scope Exclusions

- **ID:** SC-01
- **Multi-staff or multi-chair salons** — BRIEF.md's Target Users & Roles section states this directly: "a pro with staff, or a salon with several chairs, is explicitly out of scope and a different product." [RESEARCH-INFORMED: market research confirms this exclusion is well-founded rather than a missed opportunity — multi-staff/team seat management is a market-wide pattern scoped specifically to Vagaro, Booksy, and Fresha's team tiers, serving salons and multi-chair businesses, not solo operators (source: market-research.md, Feature Landscape — Differentiators, confidence: MEDIUM).]
- **ID:** SC-02
- **Any administrative or management role beyond the narrow, read-only Operator** — The persona set in user-persona.md confirms exactly three actors: the Pro, the Client, and a narrow, read-only support actor; there is no management, admin, or staff-permission role of any kind.
- **ID:** SC-03
- **Client password-style accounts or signup walls** — BRIEF.md's Target Users & Roles section states the client "must not need a password-style account just to book," and Open Questions confirms "no signup wall."
- **ID:** SC-04
- **Cross-Pro visibility of clients or bookings** — BRIEF.md is explicit that "clients' data is never visible to any other pro or client" (Target Users & Roles); no shared or aggregated view across Pros exists.

### Feature Scope Exclusions

- **ID:** SC-05
- **Instagram platform integration or API** — BRIEF.md's Ecosystem & Integrations section states Instagram is "only the place the link lives; no Instagram integration in v1," and the product is "otherwise standalone."
- **ID:** SC-06
- **Per-booking transaction fees or commission-based pricing** — BRIEF.md's Business Context is explicit: "absolutely no per-booking cut — these pros resent it and it is how they choose tools." The product's only monetization is the flat Pro Subscription & Billing feature (FEAT-17). [RESEARCH-INFORMED: market research directly corroborates this resentment — the per-new-client marketplace fee charged by StyleSeat, Booksy (opt-in), Fresha, and theCut is the space's most consistently documented professional-side complaint, most sharply around clients misclassified as "new" on repeat or referral visits (source: market-research.md, Monetization Patterns; StyleSeat and Fresha profiles, confidence: HIGH).]
- **ID:** SC-07
- **Native mobile apps or app-store distribution** — BRIEF.md's Constraints section states "no native apps, no app stores" for v1; the product is mobile-first web only.
- **ID:** SC-08
- **In-house card number storage or handling** — BRIEF.md's Constraints section is explicit that "card data is never stored or handled by the product's own code — the processor owns it"; the product never builds its own payment-storage capability.
- **ID:** SC-09
- **Multiple simultaneous locations, chairs, or timezones for a single Pro account** — BRIEF.md confirms strictly single-operator scope (Target Users & Roles); a single Pro Profile carries one timezone and one currency, not several.
- **ID:** SC-10
- **Consumer-facing client-discovery marketplace or directory** — theCut, StyleSeat, Booksy (opt-in "Boost"), and Fresha all layer a client-discovery marketplace on top of booking; the brief's whole premise is that Mara already has clients via Instagram and needs a link to convert DM-based discovery into paid, protected bookings, not a second discovery channel — and marketplace exposure is the mechanism that carries the per-new-client fee model SC-06 already excludes [AUDIT-EXCLUDED: 2 -- Competitive Feature Cross-Reference (completeness-audit.md Section 2) found this present in 4 of 5 profiled competitors as a differentiator; evaluated against BRIEF.md's link-based, standalone vision (Ecosystem & Integrations) and excluded as out of vision at any scale, not merely deferred, per market-research.md Feature Landscape — Differentiators, confidence: MEDIUM].
- **ID:** SC-11
- **Instant or expedited deposit payout for an extra fee** — theCut offers a 1.5%-fee expedited payout as a differentiator; excluded because the brief's confirmed money flow already routes the client's deposit directly into the Pro's own payout account with no platform hold (BRIEF.md, Business Context) — there is no delay for an instant-payout fee to solve, and introducing a fee-based convenience would cut against the brief's flat, no-extra-fee positioning that SC-06 protects [AUDIT-EXCLUDED: 2 -- Competitive Feature Cross-Reference found this differentiator present only in theCut and not clearly documented elsewhere (source: market-research.md, Feature Comparison Matrix, confidence: LOW-MEDIUM, single-competitor evidence) — evaluated against vision and excluded rather than added].
- **ID:** SC-12
- **AI-assisted after-hours booking or inquiry handling** — StyleSeat offers an AI assistant ("Sage") for after-hours inquiries; this is a single-competitor differentiator with no brief basis and no evidence it addresses a need this product's core loop (a client books directly against genuinely free slots, any hour) does not already solve — a client does not need an inquiry assistant when the booking page itself answers price, duration, and deposit questions directly [AUDIT-EXCLUDED: 2 -- Competitive Feature Cross-Reference found this differentiator documented for only 1 of 5 profiled competitors (source: market-research.md, Feature Comparison Matrix, confidence: LOW, single-competitor evidence); evaluated against BRIEF.md vision and excluded as added complexity without a demonstrated gap].
- **ID:** SC-13
- **Dedicated help center, contextual walkthroughs, or in-product documentation beyond guided onboarding** — Pro Onboarding & Setup (FEAT-13) already guides Mara step by step through first-run setup, and the product's design intent is to be simple enough for a non-technical solo operator to use without instruction; a separate help-content system is deliberately out of scope given the founder's three-month, solo-build timeline (BRIEF.md, Constraints) [AUDIT-EXCLUDED: 4 -- Cross-Cutting Concerns Verification (completeness-audit.md Section 4, Help and Guidance) found this concern was not mentioned anywhere in the draft; evaluated and excluded with rationale rather than silently omitted].
- **ID:** SC-14
- **Mandatory or default in-app collection of the remaining service balance** — BRIEF.md's Business Context leaves this as an open question and states "the balance after the deposit is settled in person today"; the deliberate v1 default is that the remaining balance is settled offline (cash or card reader at the chair), outside the platform — the Pro Daily Dashboard (FEAT-07) simply displays what is owed. In-App Balance Payment (FEAT-25) is offered later as an optional client-initiated alternative, not the default [AUDIT-ADDED: 1 -- Persona Journey Walkthrough's value-flow walk (completeness-audit.md Section 1) requires every open segment of a money flow to close with either a feature or an explicit ownership statement; the balance segment previously had only a Deferral Note, not an explicit statement of who owns it today].

### Scale Expectations

- **ID:** SC-15
- **A few hundred Pros in the first year, each with roughly 100–500 clients and 20–40 bookings a week** — The product is expected to stay equally responsive at this volume from MVP onward, per BRIEF.md's Scale & Non-Functional Expectations section; this is the scale the architecture must honor from day one, not a future milestone.
- **ID:** SC-16
- **Geographic expansion beyond the US (UK, Canada, Australia) is a designed-for future, not a v1 build target** — BRIEF.md states the product starts in the US and that "timezone and currency must not be hard-coded" so expansion is possible (Scale & Non-Functional Expectations); translated or locale-adapted content for these markets is genuinely out of vision until expansion is actually planned.
- **ID:** SC-17
- **Multi-year retention of client and booking history at full detail** — Expected: a Pro's client and booking history is retained in full (not summarized or purged) across several years of accumulated use, since the Booking Record & Dispute Trail (FEAT-18) explicitly depends on old records remaining intact and disputable.

## Deferral Notes

- **Client List Import** — Target phase: Later. Deferred rather than rejected; useful primarily at a Pro's initial migration moment, and the client list already builds organically from first bookings (FEAT-11) without it.
- **Cancellation Waitlist** — Target phase: Later. Deferred, resolving BRIEF.md's own open question; worth building once real cancellation volume shows how often earlier slots actually open up.
- **In-App Tipping at Checkout** — Target phase: Later. Deferred, resolving BRIEF.md's own open question on tipping; revisit once real Pros or clients ask for it.
- **In-App Balance Payment** — Target phase: Later. Deferred, resolving BRIEF.md's own open question on in-app balance settlement; the in-person default already works today via the balance-due display on the Pro Daily Dashboard (FEAT-07) and the explicit value-flow ownership statement at SC-14.
- **Recurring Appointment Booking** — Target phase: Later. Deferred, resolving BRIEF.md's own open question on standing appointments; worth adding once the single-booking loop is proven, since a recurring series introduces real complexity around individual-occurrence changes.
- **Simple Business Insights** — Target phase: Later. Deferred until Pros have accumulated enough booking history for a weekly snapshot to be meaningful rather than a near-empty view.
- **Data Export** — Target phase: Later. Deferred; a data-portability convenience with no dependency from any Core feature.
- **WhatsApp Messaging Channel** — Target phase: Later. Deferred exactly as BRIEF.md states: "WhatsApp is a later nice-to-have, not v1" (Ecosystem & Integrations).
- **Account Closure & Client Data Deletion** — Target phase: v1. Deferred from MVP because the underlying, brief-required client-record deletion right (FEAT-11) already ships at MVP; only the Pro's own full account-closure flow waits for v1, once real Pros exist who might want to leave.
- **Operator Support Console** — Target phase: v1. Deferred from MVP because the founder can troubleshoot the first handful of Pros (drawn from her own network per BRIEF.md's Business Context) directly; a dedicated read-only console — now including its own action log (FEAT-21) — becomes necessary once the Pro base grows past what she can track personally.
</content>
