---
document_type: scope-boundaries
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed
---

# Scope Boundaries

## In-Scope Summary

This product is a mobile-first booking link for one independent beauty or wellness professional, letting a client pick a service and a genuinely free time, pay a card deposit that lands directly with the pro, and receive automatic reminders — with a no-show deposit kept automatically under the pro's own cancellation policy. Core features (FEAT-01 through FEAT-12, plus the audit-added FEAT-28 Payout Account Connection & Payout Visibility and FEAT-30 Pro Booking Management) close that entire loop end to end: setup, availability, calendar sync, booking, payment, payout, messaging, cancellation policy enforcement, pro-side changes, and no-show handling. Important features (FEAT-13 through FEAT-19, plus the audit-added FEAT-27 Pro Profile & Booking Page Settings and FEAT-29 Pro Sign-In & Account Lifecycle) add the record-keeping, onboarding, profile, sign-in, consent, billing, and support machinery a real single-operator business needs from day one. Nice-to-Have features (FEAT-20 through FEAT-26) answer the brief's own open questions — waitlist, recurring appointments, in-app balance payment, tipping — plus later polish (client search, insights, WhatsApp). The product serves exactly two product roles (the Pro and the Client) plus one narrow, non-product support role. [MODIFIED: in-scope summary updated to include the four audit-added features]

## Explicit Exclusions

### User Scope Exclusions

- **ID:** SC-01
- **Multi-staff or multi-chair salon accounts** — BRIEF.md's Constraints state the product is "strictly single-operator for v1... probably forever for this product"; a pro with staff or a multi-chair salon is a different product. [RESEARCH-INFORMED: payroll, multi-staff management and per-additional-user pricing appear in four of five profiled competitors, confirming this is the salon-scale pattern the brief deliberately avoids, from the market-research Feature Comparison Matrix]
- **ID:** SC-02
- **Any administrative, manager, or staff role beyond the founder's own read-only support access** — the persona set in user-persona.md confirms exactly two product roles plus one narrow support function ("no other roles," BRIEF.md, Target Users & Roles); no additional role is grounded in the brief.
- **ID:** SC-03
- **Cross-pro or cross-client visibility** — BRIEF.md's Constraints are explicit that a client's data is "never visible to any other pro or client"; there is no shared or comparative view across accounts of any kind.
- **ID:** SC-04
- **Client accounts with passwords, or a single client profile shared across several pros** — BRIEF.md's Target Users & Roles says clients "must not face a signup wall or need a password-style account"; a client's identity is their phone number with one pro only, so booking with a second pro creates a separate, unconnected record. [AUDIT-EXCLUDED: 4 -- Roles and Sharing concern: a cross-pro client profile would share client data between pros, contradicting BRIEF.md's privacy constraint]
- **ID:** SC-05
- **Support staff acting on a Pro's behalf** — BRIEF.md limits the operator to "a read-only support view"; support cannot edit, refund, sign in as a Pro, or change a Pro's sign-in details, so a Pro who loses access to both their sign-in email and phone must regain one of them to recover. [AUDIT-EXCLUDED: 4 -- Security and Privacy Posture concern: operator-performed account changes would exceed the brief's minimal-admin boundary]

### Feature Scope Exclusions

- **ID:** SC-06
- **Instagram integration beyond serving as the destination for the bio link** — BRIEF.md's Ecosystem & Integrations states plainly: "No Instagram integration for v1." Chairtime is what the link points to, nothing more.
- **ID:** SC-07
- **Native mobile apps or app-store distribution** — BRIEF.md's Constraints specify "mobile-first web only for v1; no native apps, no app stores," matching where users already are (Instagram's in-app browser).
- **ID:** SC-08
- **Health or medical intake forms, and custom intake questionnaires generally** — BRIEF.md's Constraints explicitly exclude this: "massage/tattoo intake forms are out of scope for v1," and confirm no health-data regime applies; the only free text a client adds is a short optional note to the Pro, with a hint not to include medical information. [AUDIT-EXCLUDED: 2 -- custom intake questions are a differentiator in one competitor (Booksy); adding them would invite health data the brief keeps out of scope]
- **ID:** SC-09
- **Importing data from prior tools (DM history, paper diaries, spreadsheets)** — pros switching to Chairtime are leaving an informal, unstructured process (Instagram DMs, a paper notebook, ad hoc Venmo requests per BRIEF.md's Problem Statement); there is no structured source worth building an import path for at this scale.
- **ID:** SC-10
- **Multi-language or translated content** — BRIEF.md's Scale & Non-Functional Expectations names the US, UK, Canada, and Australia as the near-term geography, all English-language markets; no translation need is stated.
- **ID:** SC-11
- **Handling or storing card data within the product itself** — BRIEF.md's Constraints state directly: "card data is never stored or handled by the founder's code; the payment processor owns it." This is a hard boundary, not a feature choice.
- **ID:** SC-12
- **Social features, public reviews, marketplace discovery, or discoverability profiles** — the brief's entire discovery model is the pro's own Instagram presence (BRIEF.md, Ecosystem & Integrations); building a competing discovery or social layer inside Chairtime would change the product's fundamental nature. [RESEARCH-INFORMED: the two competitors with marketplace discovery fund it with a one-off 20–30% new-client commission that professionals widely resent, which BRIEF.md's "absolutely no per-booking cut" rules out, from StyleSeat and Fresha pricing-guide aggregators (HIGH confidence)]
- **ID:** SC-13
- **Card-on-file cancellation fees charged after booking** — the product protects the pro with a deposit paid up front (BRIEF.md, Vision), and charging a client's card later would require keeping a card on file for future charges. [AUDIT-EXCLUDED: 2 -- Booksy offers a card-on-file fee mode alongside deposits; excluded because the brief specifies deposits, and after-the-fact charges are exactly the disputed-charge pattern clients complain about (MEDIUM confidence)]
- **ID:** SC-14
- **Dynamic, demand-based, or time-of-day pricing** — BRIEF.md describes one price per service; varying prices by demand or time adds setup the solo pro did not ask for. [AUDIT-EXCLUDED: 2 -- "smart pricing" is an optional differentiator in one competitor, and the inability to vary price by day is a complaint about another (MEDIUM confidence); not aligned with the brief's set-up-once vision, so revisit only if pros ask for it]
- **ID:** SC-15
- **Marketing or promotional text campaigns** — clients consent to booking-related texts only (BRIEF.md, Constraints: "reminders must respect that consent"), and Chairtime sends nothing beyond confirmations, reminders, change notices, and access links. [AUDIT-EXCLUDED: 2 -- text marketing is an add-on in two competitors; excluded because it would stretch the client's booking-specific consent and add the à-la-carte complexity pros complain about]
- **ID:** SC-16
- **Card-reader hardware or in-person payment taking** — at MVP, the balance after the deposit is settled in person between the pro and the client, outside the platform, by whatever means the pro already uses; the pro simply records the booking as completed. From v1, clients may optionally pay the balance in-app (FEAT-22). [AUDIT-EXCLUDED: 1 -- value-flow walk: the balance segment at MVP is deliberately owned by the pro, off-platform; hardware billing disputes are also a reported complaint about one competitor (LOW confidence)]
- **ID:** SC-17
- **Chairtime deciding disputes between a pro and a client** — the product provides the trustworthy record (FEAT-16) and the pro can refund as goodwill (FEAT-30), but Chairtime never rules on who is right; a client who disagrees can contest the charge with their card issuer, handled through the payment processor. [AUDIT-EXCLUDED: 1 -- counterpart symmetry: dispute outcomes are owned by the pro and the payment processor's dispute process, not by the platform]
- **ID:** SC-18
- **Partial refunds and tiered cancellation schedules** — BRIEF.md's Business Context defines a binary rule (refund outside the window, kept inside it or on a no-show); partial percentages by timing are not offered in v1. [AUDIT-EXCLUDED: 1 -- value-flow walk: keeping the rule binary keeps every deposit outcome predictable for both parties]

### Scale Expectations

- **ID:** SC-19
- **Volume:** a few hundred pros in year one, each with roughly 100–500 clients and 20–40 bookings a week, honored from MVP onward — matching BRIEF.md's Scale & Non-Functional Expectations exactly; no separate scale phase-in is needed at this volume.
- **ID:** SC-20
- **Geography phase-in:** the product launches US-first at MVP; timezone and currency are treated as per-account configuration from day one (never hard-coded) so that expansion to the UK, Canada, and Australia at v1/Later requires no structural rework — per BRIEF.md's stated expansion intent. Each pro's payout account must be in the same country as their Chairtime account.
- **ID:** SC-21
- **Correctness over uptime numbers:** BRIEF.md states no specific uptime target is required, but that "booking and payment must be correct, always" and the product must "never silently double-book or lose a deposit." This bar applies at every scale from MVP onward; it is a correctness expectation, not a boundary that phases in.
- **ID:** SC-22
- **History depth:** a pro's full booking, client and deposit history is kept for as long as their account exists, so multi-year history stays available for dispute evidence and insights; after a client deletion or account closure, only de-identified financial records required by law are retained. [AUDIT-ADDED: 3 -- entity coverage: retention of Booking, Client and Deposit Transaction history was unstated]

## Deferral Notes

- **Waitlist for Cancelled Slots** — Target phase: v1. Deferred rather than rejected; it depends on Client-Initiated Cancel/Reschedule already existing and proven, and the core booking loop works without it. Worth including once cancellations are common enough to make a waitlist meaningful. [RESEARCH-INFORMED: the closest solo-focused competitor offers waitlists as a business tool (MEDIUM confidence)]
- **Recurring/Standing Appointments** — Target phase: v1. BRIEF.md's Open Questions asks directly whether this is v1 or later; deferred so the single-booking core loop is proven reliable first, given the added complexity to the availability engine. At MVP, a pro can already rebook a regular one visit at a time through Pro Booking Management.
- **In-App Balance Payment** — Target phase: v1. BRIEF.md's Open Questions leaves this open; the default in-person balance flow works without it, so it is deferred as a low-risk enhancement once deposit payment is proven in production.
- **Tipping at Checkout** — Target phase: Later. BRIEF.md's Open Questions asks where tipping fits "if anywhere"; deferred because it depends on In-App Balance Payment existing first and has no bearing on the core no-show problem.
- **Client List Search & Filter** — Target phase: v1. A new pro starts with very few clients, so this becomes valuable only as a pro's client base grows toward the brief's stated 100–500 range.
- **Booking & Revenue Insights** — Target phase: v1. Deferred until pros have enough booking history for a summary to be meaningful; not required for the core loop to deliver value.
- **WhatsApp Reminders** — Target phase: Later. BRIEF.md's Ecosystem & Integrations states this directly: "WhatsApp is a nice-to-have later, not v1."
