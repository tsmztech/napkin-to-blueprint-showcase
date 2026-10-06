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

This product is a mobile-first booking link for one independent beauty or wellness professional, letting a client pick a service and a genuinely free time, pay a card deposit, and receive automatic reminders — with a no-show deposit kept automatically under the pro's own cancellation policy. Core features (FEAT-01 through FEAT-12) close that entire loop end to end: setup, availability, calendar sync, booking, payment, messaging, cancellation policy enforcement, and no-show handling. Important features (FEAT-13 through FEAT-19) add the record-keeping, onboarding, consent, billing, and support machinery a real single-operator business needs from day one. Nice-to-Have features (FEAT-20 through FEAT-26) answer the brief's own open questions — waitlist, recurring appointments, in-app balance payment, tipping — plus later polish (client search, insights, WhatsApp). The product serves exactly two product roles (the Pro and the Client) plus one narrow, non-product support role.

## Explicit Exclusions

### User Scope Exclusions

- **ID:** SC-01
- **Multi-staff or multi-chair salon accounts** — BRIEF.md's Constraints state the product is "strictly single-operator for v1... probably forever for this product"; a pro with staff or a multi-chair salon is a different product.
- **ID:** SC-02
- **Any administrative, manager, or staff role beyond the founder's own read-only support access** — the persona set in draft-user-persona.md confirms exactly two product roles plus one narrow support function ("no other roles," BRIEF.md, Target Users & Roles); no additional role is grounded in the brief.
- **ID:** SC-03
- **Cross-pro or cross-client visibility** — BRIEF.md's Constraints are explicit that a client's data is "never visible to any other pro or client"; there is no shared or comparative view across accounts of any kind.

### Feature Scope Exclusions

- **ID:** SC-04
- **Instagram integration beyond serving as the destination for the bio link** — BRIEF.md's Ecosystem & Integrations states plainly: "No Instagram integration for v1." Chairtime is what the link points to, nothing more.
- **ID:** SC-05
- **Native mobile apps or app-store distribution** — BRIEF.md's Constraints specify "mobile-first web only for v1; no native apps, no app stores," matching where users already are (Instagram's in-app browser).
- **ID:** SC-06
- **Health or medical intake forms** — BRIEF.md's Constraints explicitly exclude this: "massage/tattoo intake forms are out of scope for v1," and confirm no health-data regime applies.
- **ID:** SC-07
- **Importing data from prior tools (DM history, paper diaries, spreadsheets)** — pros switching to Chairtime are leaving an informal, unstructured process (Instagram DMs, a paper notebook, ad hoc Venmo requests per BRIEF.md's Problem Statement); there is no structured source worth building an import path for at this scale.
- **ID:** SC-08
- **Multi-language or translated content** — BRIEF.md's Scale & Non-Functional Expectations names the US, UK, Canada, and Australia as the near-term geography, all English-language markets; no translation need is stated.
- **ID:** SC-09
- **Handling or storing card data within the product itself** — BRIEF.md's Constraints state directly: "card data is never stored or handled by the founder's code; the payment processor owns it." This is a hard boundary, not a feature choice.
- **ID:** SC-10
- **Social features, public reviews, or discoverability profiles** — the brief's entire discovery model is the pro's own Instagram presence (BRIEF.md, Ecosystem & Integrations); building a competing discovery or social layer inside Chairtime would change the product's fundamental nature.

### Scale Expectations

- **ID:** SC-11
- **Volume:** a few hundred pros in year one, each with roughly 100–500 clients and 20–40 bookings a week, honored from MVP onward — matching BRIEF.md's Scale & Non-Functional Expectations exactly; no separate scale phase-in is needed at this volume.
- **ID:** SC-12
- **Geography phase-in:** the product launches US-first at MVP; timezone and currency are treated as per-account configuration from day one (never hard-coded) so that expansion to the UK, Canada, and Australia at v1/Later requires no structural rework — per BRIEF.md's stated expansion intent.
- **ID:** SC-13
- **Correctness over uptime numbers:** BRIEF.md states no specific uptime target is required, but that "booking and payment must be correct, always" and the product must "never silently double-book or lose a deposit." This bar applies at every scale from MVP onward; it is a correctness expectation, not a boundary that phases in.

## Deferral Notes

- **Waitlist for Cancelled Slots** — Target phase: v1. Deferred rather than rejected; it depends on Client-Initiated Cancel/Reschedule already existing and proven, and the core booking loop works without it. Worth including once cancellations are common enough to make a waitlist meaningful.
- **Recurring/Standing Appointments** — Target phase: v1. BRIEF.md's Open Questions asks directly whether this is v1 or later; deferred so the single-booking core loop is proven reliable first, given the added complexity to the availability engine.
- **In-App Balance Payment** — Target phase: v1. BRIEF.md's Open Questions leaves this open; the default in-person balance flow works without it, so it is deferred as a low-risk enhancement once deposit payment is proven in production.
- **Tipping at Checkout** — Target phase: Later. BRIEF.md's Open Questions asks where tipping fits "if anywhere"; deferred because it depends on In-App Balance Payment existing first and has no bearing on the core no-show problem.
- **Client List Search & Filter** — Target phase: v1. A new pro starts with very few clients, so this becomes valuable only as a pro's client base grows toward the brief's stated 100–500 range.
- **Booking & Revenue Insights** — Target phase: v1. Deferred until pros have enough booking history for a summary to be meaningful; not required for the core loop to deliver value.
- **WhatsApp Reminders** — Target phase: Later. BRIEF.md's Ecosystem & Integrations states this directly: "WhatsApp is a nice-to-have later, not v1."
