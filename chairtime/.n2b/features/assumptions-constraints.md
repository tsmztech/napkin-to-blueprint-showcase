---
document_type: assumptions-constraints
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-21
synthesis_check: passed (2 fixes applied)
---

# Assumptions and Constraints

## Product Assumptions

### User Environment

- **ID:** ASMP-01
- **We assume clients access the booking link primarily from inside Instagram's in-app browser, on a mobile phone.** — Invalidated if: usage data shows a majority of bookings happen from a standalone mobile browser or desktop session instead, which would change what the booking experience needs to prioritize.
- **ID:** ASMP-02
- **We assume clients have a personal mobile phone number capable of receiving text messages at the time of booking.** — Invalidated if: a meaningful share of prospective clients cannot receive texts (landline-only, or a strong preference against SMS) and are unable to complete booking as a result.
- **ID:** ASMP-03
- **We assume Pros primarily manage their business from a mobile phone, checking the product in short bursts between clients rather than in extended desk sessions.** — Invalidated if: usage data shows Pros predominantly use the product from a desktop or tablet session instead.
- **ID:** ASMP-04
- **We assume Pros already maintain a personal Google or Apple Calendar they are willing to connect.** — Invalidated if: a significant share of Pros have no external personal calendar at all, making two-way sync irrelevant to a large part of the user base.

<!-- [INFERRED: carried from Visionary draft] -->

### User Behavior

- **ID:** ASMP-05
- **We assume clients are willing to pay a card deposit at the time of booking, before meeting the Pro in person, for services roughly in the $20–$150 range.** — Invalidated if: a meaningful share of prospective clients abandon the flow specifically at the deposit-payment step rather than completing it.
- **ID:** ASMP-06
- **We assume Pros will set a deposit amount and cancellation window strict enough to meaningfully deter no-shows, without setting it so strict that it deters bookings altogether.** — Invalidated if: no-show rates remain materially unchanged after adoption, or booking volume drops sharply once a Pro tightens their policy.
- **ID:** ASMP-07
- **We assume clients will engage with a two-day-before reminder rather than ignoring it.** — Invalidated if: reminder response rates stay persistently low even after a client has used the product for multiple bookings with the same Pro.

<!-- [INFERRED: carried from Visionary draft] -->

### Product Context

- **ID:** ASMP-08
- **We assume this is a standalone product used directly by the Pro, not embedded inside or resold through another salon-management or scheduling platform.** — Invalidated if: the founder's distribution model shifts toward reselling through a third-party platform rather than direct Pro adoption.
- **ID:** ASMP-09
- **We assume the earliest Pros come from the founder's own personal network and subsequent peer referrals, not broad paid marketing.** — Invalidated if: the actual growth pattern in the first months is dominated by paid acquisition rather than personal and referral channels (BRIEF.md, Business Context).
- **ID:** ASMP-10
- **We assume prospective Pros will value a flat, no-marketplace-fee subscription enough to switch from free-but-limited tools or DM-based booking, without needing marketplace-driven client discovery in exchange.** — Invalidated if: adoption data or direct feedback shows prospective Pros hold out for marketplace-style client discovery as much as they want fee avoidance, making the fee-free positioning alone insufficient to win switches. [RESEARCH-INFORMED: market research shows every marketplace-style competitor's per-new-client fee draws sustained complaint, while GlossGenius — the one fee-free competitor profiled — does not offer marketplace discovery in exchange; no profiled competitor combines a flat, no-commission model with marketplace-style client discovery (source: market-research.md, Insights for This Product, confidence: MEDIUM). This assumption makes explicit and testable the business-model bet the brief's monetization choice already implies, rather than leaving it implicit.]

## Product Constraints

- **ID:** ASMP-11
- **Mobile-first web only, no native apps or app stores** — BRIEF.md's Constraints section states this directly for v1; every design and feature decision prioritizes a mobile web experience that works well inside Instagram's in-app browser.
- **ID:** ASMP-12
- **Flat monthly subscription only, no per-booking fee** — A deliberate monetization choice per BRIEF.md's Business Context: "absolutely no per-booking cut... it is how these pros choose tools." This is a product positioning constraint, not a pricing detail left open for later.
- **ID:** ASMP-13
- **Single-operator scope only** — BRIEF.md's Target Users & Roles section is explicit that multi-staff or multi-chair support is "a different product," not a future expansion of this one; this is a deliberate scope boundary, not a current technical limitation.
- **ID:** ASMP-14
- **No password-style client accounts** — A deliberate identity-design choice: phone-number verification is the permanent, chosen identity mechanism for clients, not a placeholder for a future login system, per BRIEF.md's stated "no signup wall."

<!-- [INFERRED: carried from Visionary draft] -->

## Non-Functional Expectations

- **ID:** ASMP-15
- **Responsiveness: the booking page, live availability, and payment steps feel instant on a typical mobile connection inside Instagram's in-app browser — free slots render within about 1 second, and the full booking-to-confirmation flow completes in under a minute.** — Basis: BRIEF.md's Scale & Non-Functional Expectations section states the product "must work well" inside Instagram's in-app browser, and The Experience section states the full flow takes "under a minute."
- **ID:** ASMP-16
- **Data volume and growth: each Pro accumulates roughly 100–500 clients and 20–40 bookings a week, with a few hundred Pros active in the first year; the product stays equally responsive as multi-year booking and client history accumulates.** — Basis: BRIEF.md's Scale & Non-Functional Expectations section states these figures directly.
- **ID:** ASMP-17
- **Privacy posture: a client's data is visible only to the one Pro it belongs to, is never shared across Pros, and a Pro can permanently delete a client's record on request.** — Basis: BRIEF.md's Constraints and Target Users & Roles sections.
- **ID:** ASMP-18
- **Reliability posture: the product must never silently double-book a slot or lose a deposit — this quality bar is prioritized above adding new features.** — Basis: BRIEF.md's Scale & Non-Functional Expectations section states this explicitly as the product's non-negotiable quality bar. [RESEARCH-INFORMED: market research shows no profiled competitor markets this as an explicit, headline guarantee despite double-booking- and deposit-adjacent complaints appearing across the category's reviews (source: market-research.md, Feature Landscape — Absent Features, confidence: derived/qualitative), reinforcing this as a genuine differentiator worth holding to a strict internal bar.]
- **ID:** ASMP-19
- **Compliance: explicit client consent is captured before any text message beyond the initial identity-verification code is sent, reminders respect a later withdrawal of that consent — including a standard STOP-reply opt-out mechanism — and messaging behavior is designed to respect US texting regulation.** — Basis: BRIEF.md's Constraints section, "Messaging (regulatory)." [AUDIT-ADDED: 3 -- Entity Coverage Verification's inverse check found no feature specified how consent withdrawal actually happens; product-features.md FEAT-16 now specifies a STOP-reply mechanism, reflected here as the compliance basis it satisfies.]
- **ID:** ASMP-20
- **Compliance: card payment data is never captured, transmitted through, or stored by the product's own code at any point.** — Basis: BRIEF.md's Constraints section, "Payments (regulatory / security)."
- **ID:** ASMP-21
- **Accessibility baseline: legible text, adequate touch-target sizing, and screen-reader-compatible labeling are expected across both the client booking flow and the Pro's dashboard, since neither a first-time client tapping through Instagram's in-app browser nor a Pro glancing at her phone between clients should be excluded by a text-heavy or touch-precision-dependent design.** — Basis: [AUDIT-ADDED: 4 -- Cross-Cutting Concerns Verification (completeness-audit.md Section 4 and decomposition-checklists.md Section 2, Accessibility Baseline) found no product-level accessibility decision anywhere in the draft set. No dedicated accessibility audit or formal compliance certification (e.g., WCAG conformance testing) is targeted in v1, given the founder's three-month, solo-build timeline (BRIEF.md, Constraints); this is a deliberate baseline decision, not a silent gap.]

## Dependencies

- **ID:** ASMP-22
- **Payment-processing capability** — Required for both client deposit collection and the Pro's own subscription billing. Without it, neither money flow BRIEF.md describes (deposits into the Pro's payout account, subscription charges from the Pro) can exist, and Deposit Payment at Booking (FEAT-04) and Pro Subscription & Billing (FEAT-17) cannot function.
- **ID:** ASMP-23
- **Transactional text-messaging delivery capability** — Required for identity verification at booking, instant booking confirmations, and timed reminders, including processing inbound STOP-style opt-out replies. Without it, Client Identity & Booking Details Capture (FEAT-03) and Booking Confirmation & Reminders (FEAT-05) cannot function, and the product's core communications loop collapses back into manual texting.
- **ID:** ASMP-24
- **Two-way calendar-sync capability with the Pro's personal calendar provider** — Required for Two-Way Calendar Sync (FEAT-12). Without it, only internally-known bookings and manual blocks (FEAT-10) protect availability, and a Pro's external personal commitments could silently create a double-booking risk.
- **ID:** ASMP-25
- **Modest, fixed-cost infrastructure that fits within roughly $100/month until the product earns revenue** — BRIEF.md's Constraints section states this budget ceiling explicitly. This shapes what scale of technical approach Stage 4 can responsibly recommend before the product has paying Pros to fund heavier infrastructure.

<!-- [INFERRED: carried from Visionary draft, except ASMP-19 and ASMP-21 as marked above] -->
</content>
