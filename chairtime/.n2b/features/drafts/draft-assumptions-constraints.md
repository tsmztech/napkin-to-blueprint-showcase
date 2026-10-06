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
- **We assume clients access the booking page primarily from inside Instagram's in-app browser on a phone.** Invalidated if: usage data shows a majority of client sessions arrive from outside Instagram (direct link shares, other social platforms), which would change which in-app-browser accommodations matter most.
- **ID:** ASMP-02
- **We assume pros run their day-to-day schedule from a phone, reserving desktop for one-time setup only.** Invalidated if: usage data shows pros regularly running their live daily schedule from a desktop browser during working hours.
- **ID:** ASMP-03
- **We assume both pros and clients have reliable, if intermittent, mobile data access at the moments they use the product.** Invalidated if: a meaningful share of target pros or clients regularly operate in low- or no-connectivity settings during booking or check-in, which would require rethinking the product's deliberate online-only correctness stance.

### User Behavior

- **ID:** ASMP-04
- **We assume clients are willing to pay a card deposit to an individual businessperson they found on Instagram, without an established brand behind the request.** Invalidated if: booking-funnel data shows a large share of clients abandon specifically at the deposit-payment step, citing distrust of the individual pro rather than price or friction.
- **ID:** ASMP-05
- **We assume pros will set their own cancellation policy in a way that prevents most no-shows without generating frequent client disputes.** Invalidated if: dispute rates (per the Policy Clarity at Booking metric) stay persistently high across pros, suggesting policies are being set in ways clients don't understand or accept at booking time.
- **ID:** ASMP-06
- **We assume a pro checks Chairtime in short, reactive bursts between clients rather than in one planning session per day.** Invalidated if: usage data shows pros primarily reviewing their schedule once per day rather than throughout the day.

### Product Context

- **ID:** ASMP-07
- **We assume this product remains a standalone booking-and-deposit tool, not a broader salon or business-management suite.** Invalidated if: founder or pro feedback reveals sustained demand for adjacent capabilities (inventory, staff payroll, multi-service business management) that only make sense for a multi-person business — which would contradict the strictly single-operator positioning.
- **ID:** ASMP-08
- **We assume Instagram remains the pros' primary client-discovery channel throughout the period this blueprint covers.** Invalidated if: target pros shift their client-discovery activity to a different platform in large numbers, which would change where the booking link needs to live and how it's shared.

## Product Constraints

- **ID:** ASMP-09
- **Single-operator only** — BRIEF.md's Constraints state this is deliberate and permanent ("probably forever for this product"), not a phased limitation to be lifted later; multi-staff and multi-chair scheduling are a different product entirely.
- **ID:** ASMP-10
- **Flat monthly subscription, no per-booking fee** — a deliberate business-model constraint per BRIEF.md's Business Context: pros "resent" per-booking cuts, and it is described as "how they choose tools." This shapes pricing and billing design, not just a default.
- **ID:** ASMP-11
- **No card data ever touches the product's own code** — a deliberate security and regulatory constraint per BRIEF.md's Constraints ("I never want to see or store a card number"); all card handling is delegated entirely to the payment-processing capability.
- **ID:** ASMP-12
- **Mobile-first web only, no native apps** — a deliberate platform constraint per BRIEF.md's Constraints, matching where users already are (phone, Instagram in-app browser) and the founder's speed-to-launch goal.
- **ID:** ASMP-13
- **The product enforces whatever cancellation policy the pro sets; it never recommends or overrides it** — a deliberate constraint keeping business judgment with the pro rather than the platform, consistent with the brief's framing of the policy as the pro's own rule that clients agree to.

## Non-Functional Expectations

- **ID:** ASMP-14
- **Responsiveness: available slots appear within roughly one second of a service selection, and a full booking (selection through paid confirmation) completes in under one minute.** — Basis: BRIEF.md's Vision states the one-minute booking benchmark directly, and its Scale & Non-Functional Expectations section makes correctness and speed the product's defining quality bar.
- **ID:** ASMP-15
- **Data volume and growth: a few hundred pros in year one, each with roughly 100–500 clients and 20–40 bookings a week, with the product staying equally responsive as pros accumulate history over multiple years.** — Basis: BRIEF.md's Scale & Non-Functional Expectations, stated directly.
- **ID:** ASMP-16
- **Privacy posture: a client's data is visible only to their own pro and to themselves; a pro can permanently delete a client's record on request.** — Basis: BRIEF.md's Privacy and Constraints sections, stated directly.
- **ID:** ASMP-17
- **Compliance: US SMS-consent rules apply to all client texting (explicit opt-in captured at booking, honored immediately on opt-out); no health-data regime applies, since intake and clinical data are explicitly out of scope.** — Basis: BRIEF.md's Constraints, "Regulated-domain confirmation" section, stated directly.
- **ID:** ASMP-18
- **Geography and localization: timezone and currency are per-account configuration from day one, never hard-coded, so expansion beyond the US requires no structural rework.** — Basis: BRIEF.md's Scale & Non-Functional Expectations: "the UK, Canada and Australia are the obvious next markets, so timezone and currency must not be hard-coded."
- **ID:** ASMP-19
- **Reliability is expressed as a correctness bar, not a numeric uptime target: the product must never silently double-book a slot or lose a deposit.** — Basis: BRIEF.md's Scale & Non-Functional Expectations states directly that "no specific uptime number was given" and frames correctness, not uptime percentage, as the requirement.

## Dependencies

- **ID:** ASMP-20
- **Payment-processing capability** — required to take client deposits, pay them out to the pro, and bill the pro's own monthly subscription. Without it, the product has no way to collect money or generate revenue at all; BRIEF.md's Constraints require that this capability, not the product's own code, owns all card data.
- **ID:** ASMP-21
- **Transactional text-messaging capability, with email as a fallback channel** — required to send booking confirmations and pre-appointment reminders. Without it, the product cannot deliver the automatic-reminder promise that replaces the pro's manual texting habit; BRIEF.md's Ecosystem & Integrations names texting as the v1 channel with email as an acceptable fallback.
- **ID:** ASMP-22
- **Calendar-sync capability (reading and writing to a pro's personal calendar)** — required for the two-way sync described in BRIEF.md's Ecosystem & Integrations. Without it, the availability engine cannot account for a pro's real-world commitments outside Chairtime, directly threatening the "never double-book" correctness bar.
- **ID:** ASMP-23
- **A searchable, per-pro record store for services, bookings, clients, and payment outcomes** — required for the product to function at all across sessions; without persistent, per-account data, nothing booked, paid, or configured could be relied on the next time either the pro or the client returns.
