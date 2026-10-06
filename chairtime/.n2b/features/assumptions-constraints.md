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
- **ID:** ASMP-07
- **We assume most clients will opt in to text messages when they book.** Invalidated if: fewer than half of clients opt in to texts across pros, which would make email the main reminder channel and weaken the one-tap reminder experience the Reminder Response Rate metric depends on. [AUDIT-ADDED: 1 -- journey walk: the email fallback added to the booking flow is only a fallback if most clients choose texts]
- **ID:** ASMP-08
- **We assume pros will complete the payment processor's identity and bank verification during setup without help.** Invalidated if: more than 1 in 5 pros who reach the payout step have not finished verification within a week, which would make payout setup the main barrier to a live booking link. [AUDIT-ADDED: 1 -- value-flow walk: the audit-added payout-account step now gates the booking link going live]

### Product Context

- **ID:** ASMP-09
- **We assume this product remains a standalone booking-and-deposit tool, not a broader salon or business-management suite.** Invalidated if: founder or pro feedback reveals sustained demand for adjacent capabilities (inventory, staff payroll, multi-service business management) that only make sense for a multi-person business — which would contradict the strictly single-operator positioning.
- **ID:** ASMP-10
- **We assume Instagram remains the pros' primary client-discovery channel throughout the period this blueprint covers.** Invalidated if: target pros shift their client-discovery activity to a different platform in large numbers, which would change where the booking link needs to live and how it's shared.
- **ID:** ASMP-11
- **We assume the MVP scope — including two-way sync with both Google and Apple calendars and the audit-added payout, sign-in, profile and pro-side booking features — can reach a first paying pro within about three months for a solo founder building with AI coding tools.** Invalidated if: by the end of month two the core loop (book, pay deposit, remind, cancel/refund, no-show) does not yet work end to end, which would call for re-phasing lower-risk MVP items with the founder rather than cutting the correctness bar. [AUDIT-ADDED: 4 -- BRIEF.md's Constraints set a three-month timeline that the final 23-feature MVP must be checked against]
- **ID:** ASMP-12
- **We assume a solo pro's clients are willing to pay each deposit fresh rather than keep a card on file with the pro.** Invalidated if: a meaningful share of repeat clients abandon rebooking at the deposit step, citing re-entering card details, which would argue for a processor-held saved-card option later. [RESEARCH-INFORMED: competitors lean on card-on-file mechanisms, but difficulty removing stored client cards is a frequently mentioned complaint about one of them (MEDIUM confidence), supporting BRIEF.md's no-stored-card posture]

## Product Constraints

- **ID:** ASMP-13
- **Single-operator only** — BRIEF.md's Constraints state this is deliberate and permanent ("probably forever for this product"), not a phased limitation to be lifted later; multi-staff and multi-chair scheduling are a different product entirely.
- **ID:** ASMP-14
- **Flat monthly subscription, no per-booking fee** — a deliberate business-model constraint per BRIEF.md's Business Context: pros "resent" per-booking cuts, and it is described as "how they choose tools." This shapes pricing and billing design, not just a default.
- **ID:** ASMP-15
- **No card data ever touches the product's own code** — a deliberate security and regulatory constraint per BRIEF.md's Constraints ("I never want to see or store a card number"); all card handling is delegated entirely to the payment-processing capability.
- **ID:** ASMP-16
- **Mobile-first web only, no native apps** — a deliberate platform constraint per BRIEF.md's Constraints, matching where users already are (phone, Instagram in-app browser) and the founder's speed-to-launch goal.
- **ID:** ASMP-17
- **The product enforces whatever cancellation policy the pro sets; it offers a common default as a starting point during setup but never overrides a policy the pro has chosen** — a deliberate constraint keeping business judgment with the pro rather than the platform, consistent with the brief's framing of the policy as the pro's own rule that clients agree to. [MODIFIED: "never recommends" softened to "offers a common default as a starting point" because the onboarding wizard (FEAT-15) proposes a default window, which the draft constraint contradicted -- synthesis check fix]
- **ID:** ASMP-18
- **Money never rests with the platform** — client deposits (and, from v1, balances and tips) go from the client's card straight to the pro's own payout account through the payment processor; Chairtime takes no cut and never holds or routes funds itself. A deliberate business-model and trust constraint per BRIEF.md's Business Context ("the platform takes no cut of any of it"). [AUDIT-ADDED: 1 -- value-flow walk: the path of every unit of money needed to be stated as a product rule]
- **ID:** ASMP-19
- **Deposit outcomes are binary and symmetric** — outside the window a client's cancellation is refunded in full, inside it (or on a no-show) the deposit is kept, and any cancellation made by the pro is always refunded in full. A deliberate fairness constraint extending BRIEF.md's Business Context to the case where the pro is the party who cancels. [AUDIT-ADDED: 1 -- counterpart symmetry]
- **ID:** ASMP-20
- **Support access stays read-only and visible** — the operator can look but never change anything, and every look is recorded in the pro's own account activity. A deliberate constraint per BRIEF.md's Target Users & Roles ("minimal admin access"). [AUDIT-ADDED: 4 -- Audit Logging concern]

## Non-Functional Expectations

- **ID:** ASMP-21
- **Responsiveness: available slots appear within roughly one second of a service selection, and a full booking (selection through paid confirmation) completes in under one minute.** — Basis: BRIEF.md's Vision states the one-minute booking benchmark directly, and its Scale & Non-Functional Expectations section makes correctness and speed the product's defining quality bar.
- **ID:** ASMP-22
- **Data volume and growth: a few hundred pros in year one, each with roughly 100–500 clients and 20–40 bookings a week, with the product staying equally responsive as pros accumulate history over multiple years.** — Basis: BRIEF.md's Scale & Non-Functional Expectations, stated directly.
- **ID:** ASMP-23
- **Privacy posture: a client's data is visible only to their own pro and to themselves; a pro can permanently delete a client's record on request.** — Basis: BRIEF.md's Privacy and Constraints sections, stated directly.
- **ID:** ASMP-24
- **Compliance: US SMS-consent rules apply to all client texting (explicit opt-in captured at booking, honored immediately on opt-out); no health-data regime applies, since intake and clinical data are explicitly out of scope.** — Basis: BRIEF.md's Constraints, "Regulated-domain confirmation" section, stated directly.
- **ID:** ASMP-25
- **Geography and localization: timezone and currency are per-account configuration from day one, never hard-coded, so expansion beyond the US requires no structural rework.** — Basis: BRIEF.md's Scale & Non-Functional Expectations: "the UK, Canada and Australia are the obvious next markets, so timezone and currency must not be hard-coded."
- **ID:** ASMP-26
- **Reliability is expressed as a correctness bar, not a numeric uptime target: the product must never silently double-book a slot or lose a deposit.** — Basis: BRIEF.md's Scale & Non-Functional Expectations states directly that "no specific uptime number was given" and frames correctness, not uptime percentage, as the requirement. [RESEARCH-INFORMED: glitches and crashes in the core scheduling workflow are reported for two competitors at comparable booking volumes (MEDIUM confidence), so correctness under everyday use is a documented market gap]
- **ID:** ASMP-27
- **Offline and loading posture: anything that books, pays, cancels, refunds or marks a no-show needs a live connection and says so plainly when it is missing; the pro's most recently loaded schedule, client list and money list stay readable offline; every screen that waits shows an in-place indicator rather than a blank page, and nothing appears tappable before real data has loaded.** — Basis: BRIEF.md's correctness bar ("never silently double-book or lose a deposit") and the decomposition checklist's Offline and Loading items; decided product-wide so every feature's States field follows one rule. [AUDIT-ADDED: 4 -- Offline/Degraded and Loading concerns needed a product-level decision]
- **ID:** ASMP-28
- **Accessibility baseline: every client and pro screen is readable and fully operable at phone width inside a social-media in-app browser, with text that scales, sufficient contrast, controls large enough to tap reliably, and full use by screen-reader users; nothing relies on color alone (for example, the paid badge also carries a word).** — Basis: BRIEF.md's Scale & Non-Functional Expectations (mobile-first, Instagram in-app browser) and the decomposition checklist's Accessibility item. [AUDIT-ADDED: 4 -- Accessibility baseline was not decided in the draft]
- **ID:** ASMP-29
- **Messaging timing: automatic reminders reach clients only during reasonable daytime hours (roughly 8am–9pm in the pro's timezone), and confirmations arrive within about a minute of payment.** — Basis: BRIEF.md's Constraints ("reminders must respect that consent," citing US texting rules) and its Vision ("a confirmation text lands immediately"). [AUDIT-ADDED: 4 -- Compliance concern]
- **ID:** ASMP-30
- **Account protection: a pro's account, which holds every client's contact details, is protected by a one-time-code sign-in with new-device alerts; client access links are short-lived and open only that client's bookings with that one pro.** — Basis: BRIEF.md's Privacy section ("a client's data is visible only to their pro") and the decomposition checklist's Security and Privacy Posture item. [AUDIT-ADDED: 4 -- Security and Privacy Posture concern]

## Dependencies

- **ID:** ASMP-31
- **Payment-processing capability** — required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription. Without it, the product has no way to collect money or generate revenue at all; BRIEF.md's Constraints require that this capability, not the product's own code, owns all card data. [MODIFIED: connected payout accounts, identity verification, refunds and dispute notifications named explicitly after the value-flow audit]
- **ID:** ASMP-32
- **Transactional text-messaging capability, with email as a fallback channel** — required to send booking confirmations and pre-appointment reminders. Without it, the product cannot deliver the automatic-reminder promise that replaces the pro's manual texting habit; BRIEF.md's Ecosystem & Integrations names texting as the v1 channel with email as an acceptable fallback.
- **ID:** ASMP-33
- **Calendar-sync capability (reading and writing to a pro's personal calendar)** — required for the two-way sync described in BRIEF.md's Ecosystem & Integrations. Without it, the availability engine cannot account for a pro's real-world commitments outside Chairtime, directly threatening the "never double-book" correctness bar.
- **ID:** ASMP-34
- **A searchable, per-pro record store for services, bookings, clients, and payment outcomes** — required for the product to function at all across sessions; without persistent, per-account data, nothing booked, paid, or configured could be relied on the next time either the pro or the client returns.
- **ID:** ASMP-35
- **File storage capability for pro profile photos** — required for the photo shown on the booking page (FEAT-27). Without it, the booking page shows the pro's name only; the booking loop itself is unaffected. [AUDIT-ADDED: 3 -- the profile photo captured by the audit-added Pro Profile & Booking Page Settings needs somewhere to live]
