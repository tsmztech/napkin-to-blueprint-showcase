---
document_type: assumptions-constraints
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-21
coherence_check: passed
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

### User Behavior

- **ID:** ASMP-05
- **We assume clients are willing to pay a card deposit at the time of booking, before meeting the Pro in person, for services roughly in the $20–$150 range.** — Invalidated if: a meaningful share of prospective clients abandon the flow specifically at the deposit-payment step rather than completing it.
- **ID:** ASMP-06
- **We assume Pros will set a deposit amount and cancellation window strict enough to meaningfully deter no-shows, without setting it so strict that it deters bookings altogether.** — Invalidated if: no-show rates remain materially unchanged after adoption, or booking volume drops sharply once a Pro tightens their policy.
- **ID:** ASMP-07
- **We assume clients will engage with a two-day-before reminder rather than ignoring it.** — Invalidated if: reminder response rates stay persistently low even after a client has used the product for multiple bookings with the same Pro.

### Product Context

- **ID:** ASMP-08
- **We assume this is a standalone product used directly by the Pro, not embedded inside or resold through another salon-management or scheduling platform.** — Invalidated if: the founder's distribution model shifts toward reselling through a third-party platform rather than direct Pro adoption.
- **ID:** ASMP-09
- **We assume the earliest Pros come from the founder's own personal network and subsequent peer referrals, not broad paid marketing.** — Invalidated if: the actual growth pattern in the first months is dominated by paid acquisition rather than personal and referral channels (BRIEF.md, Business Context).

## Product Constraints

- **ID:** ASMP-10
- **Mobile-first web only, no native apps or app stores** — BRIEF.md's Constraints section states this directly for v1; every design and feature decision prioritizes a mobile web experience that works well inside Instagram's in-app browser.
- **ID:** ASMP-11
- **Flat monthly subscription only, no per-booking fee** — A deliberate monetization choice per BRIEF.md's Business Context: "absolutely no per-booking cut... it is how these pros choose tools." This is a product positioning constraint, not a pricing detail left open for later.
- **ID:** ASMP-12
- **Single-operator scope only** — BRIEF.md's Target Users & Roles section is explicit that multi-staff or multi-chair support is "a different product," not a future expansion of this one; this is a deliberate scope boundary, not a current technical limitation.
- **ID:** ASMP-13
- **No password-style client accounts** — A deliberate identity-design choice: phone-number verification is the permanent, chosen identity mechanism for clients, not a placeholder for a future login system, per BRIEF.md's stated "no signup wall."

## Non-Functional Expectations

- **ID:** ASMP-14
- **Responsiveness: the booking page, live availability, and payment steps feel instant on a typical mobile connection inside Instagram's in-app browser — free slots render within about 1 second, and the full booking-to-confirmation flow completes in under a minute.** — Basis: BRIEF.md's Scale & Non-Functional Expectations section states the product "must work well" inside Instagram's in-app browser, and The Experience section states the full flow takes "under a minute."
- **ID:** ASMP-15
- **Data volume and growth: each Pro accumulates roughly 100–500 clients and 20–40 bookings a week, with a few hundred Pros active in the first year; the product stays equally responsive as multi-year booking and client history accumulates.** — Basis: BRIEF.md's Scale & Non-Functional Expectations section states these figures directly.
- **ID:** ASMP-16
- **Privacy posture: a client's data is visible only to the one Pro it belongs to, is never shared across Pros, and a Pro can permanently delete a client's record on request.** — Basis: BRIEF.md's Constraints and Target Users & Roles sections.
- **ID:** ASMP-17
- **Reliability posture: the product must never silently double-book a slot or lose a deposit — this quality bar is prioritized above adding new features.** — Basis: BRIEF.md's Scale & Non-Functional Expectations section states this explicitly as the product's non-negotiable quality bar.
- **ID:** ASMP-18
- **Compliance: explicit client consent is captured before any text message beyond the initial identity-verification code is sent, reminders respect a later withdrawal of that consent, and messaging behavior is designed to respect US texting regulation.** — Basis: BRIEF.md's Constraints section, "Messaging (regulatory)."
- **ID:** ASMP-19
- **Compliance: card payment data is never captured, transmitted through, or stored by the product's own code at any point.** — Basis: BRIEF.md's Constraints section, "Payments (regulatory / security)."

## Dependencies

- **ID:** ASMP-20
- **Payment-processing capability** — Required for both client deposit collection and the Pro's own subscription billing. Without it, neither money flow BRIEF.md describes (deposits into the Pro's payout account, subscription charges from the Pro) can exist, and Deposit Payment at Booking (FEAT-04) and Pro Subscription & Billing (FEAT-17) cannot function.
- **ID:** ASMP-21
- **Transactional text-messaging delivery capability** — Required for identity verification at booking, instant booking confirmations, and timed reminders. Without it, Client Identity & Booking Details Capture (FEAT-03) and Booking Confirmation & Reminders (FEAT-05) cannot function, and the product's core communications loop collapses back into manual texting.
- **ID:** ASMP-22
- **Two-way calendar-sync capability with the Pro's personal calendar provider** — Required for Two-Way Calendar Sync (FEAT-12). Without it, only internally-known bookings and manual blocks (FEAT-10) protect availability, and a Pro's external personal commitments could silently create a double-booking risk.
- **ID:** ASMP-23
- **Modest, fixed-cost infrastructure that fits within roughly $100/month until the product earns revenue** — BRIEF.md's Constraints section states this budget ceiling explicitly. This shapes what scale of technical approach Stage 4 can responsibly recommend before the product has paying Pros to fund heavier infrastructure.
