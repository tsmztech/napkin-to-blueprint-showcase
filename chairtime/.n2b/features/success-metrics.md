---
document_type: success-metrics
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-21
synthesis_check: passed (1 fix applied)
---

# Success Metrics

## Summary

This document contains 26 success metrics covering all 17 Core features and all 4 Important features: 19 product-experience metrics, 4 adoption/engagement/business KPIs, and 3 user-facing performance expectations. Together they trace directly back to the brief's own Success Criteria — no unpaid no-shows, no DMs, no double bookings, no lost deposits, and peer-driven growth. Market research corroborates several of these as genuine market differentiators rather than only internal quality bars: no profiled competitor markets double-booking prevention or fully-enforced deposit/no-show integrity as a headline guarantee, despite documented enforcement gaps appearing across the category (market-research.md, Feature Landscape — Absent Features). No new metrics were added or removed during synthesis; five received [RESEARCH-INFORMED] enrichment.

---

### Booking Page Clarity

**Description:** Measures whether a client can understand what they're booking and what it costs, without any explanation from Mara, before they proceed.

**Target:** A client can name the service, price, duration, and deposit rule correctly after viewing the booking page, without asking Mara a clarifying question first.

**Rationale:** The brief's entire opening promise depends on this page replacing a DM conversation entirely — if clients still need to ask Mara questions, the DM negotiation the product exists to remove has simply moved. [INFERRED: carried from Visionary draft]

**Persona:** Taylor

**Connected Feature:** Public Booking Page

---

### Zero Double-Booking Incidents

**Description:** Measures whether the availability shown to a client ever results in two bookings for the same, overlapping time.

**Target:** Zero double bookings occur across all Pros, at any usage volume within the stated scale (a few hundred Pros, 20–40 bookings a week each).

**Rationale:** BRIEF.md names this the product's non-negotiable quality bar: "the moment it silently double-books... the pro is gone and tells their friends" (Scale & Non-Functional Expectations). [RESEARCH-INFORMED: no profiled competitor markets double-booking prevention as an explicit, headline reliability guarantee despite double-booking-adjacent complaints appearing in review evidence across the category (source: market-research.md, Feature Landscape — Absent Features, confidence: derived/qualitative) — making a zero-incident bar a genuine market differentiator, not merely an internal target.]

**Persona:** All

**Connected Feature:** Live Availability & Slot Booking

---

### Booking Completion Speed

**Description:** Measures how quickly a client can go from tapping the bio link to a confirmed, paid booking.

**Target:** A client completes name/phone entry, verification, and consent in under 30 seconds, contributing to the brief's overall "under a minute" total booking time.

**Rationale:** BRIEF.md's Experience section states the whole flow — link tap to done — should take "under a minute"; identity capture is one of the few steps requiring the client to type anything, so it is the most likely source of friction. [INFERRED: carried from Visionary draft]

**Persona:** Taylor

**Connected Feature:** Client Identity & Booking Details Capture

---

### Zero Lost Deposits

**Description:** Measures whether a client's deposit payment, once captured, is ever lost, misapplied to the wrong booking, or unaccounted for.

**Target:** Zero instances of a captured deposit not being correctly reflected against its booking, across all Pros.

**Rationale:** BRIEF.md names deposit integrity, alongside double-booking integrity, as the product's non-negotiable quality bar (Scale & Non-Functional Expectations) and as a headline Success Criterion: "nobody has ever had... a lost deposit." [RESEARCH-INFORMED: this targets exactly the reliability gap the market has not solved — every profiled competitor with reviewable enforcement evidence (GlossGenius, Booksy, Fresha) has at least one documented case of a deposit or no-show charge failing silently or without notification (source: market-research.md, Common Complaint Themes, confidence: HIGH).]

**Persona:** All

**Connected Feature:** Deposit Payment at Booking

---

### Reminder Response Rate

**Description:** Measures what share of clients respond to their two-day-before reminder with either "I'll be there" or "I need to reschedule," rather than leaving it unanswered.

**Target:** At least 70% of reminders receive an explicit one-tap response before the appointment.

**Rationale:** The brief's reminder mechanism only replaces Mara's manual texting if clients actually engage with it; an unanswered reminder gives Mara no more certainty than she had before switching. [INFERRED: carried from Visionary draft]

**Persona:** Taylor

**Connected Feature:** Booking Confirmation & Reminders

---

### Self-Service Reschedule Rate

**Description:** Measures what share of client-initiated plan changes are completed entirely within the product, without a phone call or DM to Mara.

**Target:** At least 90% of client reschedules or cancellations happen through the self-service flow rather than an out-of-product message to Mara.

**Rationale:** BRIEF.md's success criterion "pros stop taking bookings by DM entirely" extends naturally to plan changes — if clients still DM Mara to reschedule, the product hasn't fully replaced the old behavior. [INFERRED: carried from Visionary draft]

**Persona:** Taylor

**Connected Feature:** Client Self-Service Reschedule & Cancellation

---

### Daily Dashboard Glanceability

**Description:** Measures whether Mara can understand her remaining day — who's paid, any notes, what's owed — in a single glance between clients, without scrolling or digging.

**Target:** Mara can identify the paid status, note, and balance due for her next booking within about 3 seconds of opening the dashboard.

**Rationale:** BRIEF.md describes this exact moment: "the pro glances at their phone between clients" (The Experience) — the feature fails its purpose if it requires sustained attention. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Pro Daily Dashboard

---

### No Unpaid No-Shows

**Description:** Measures whether a client who fails to show up ever results in Mara losing the deposit she was owed.

**Target:** 100% of marked no-shows result in the deposit being retained, with zero instances of an unpaid no-show going uncompensated.

**Rationale:** This is BRIEF.md's headline Success Criterion, stated in the Pro's own words: "I haven't had an unpaid no-show since I switched" (Success Criteria). [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** No-Show & Cancellation Deposit Handling

---

### Policy Setup Completeness

**Description:** Measures whether a new Pro finishes onboarding with a complete, usable policy — service, price, deposit rule, hours, and cancellation window all set — rather than a partial configuration that blocks bookings.

**Target:** At least 95% of Pros who reach the bio-link step have a fully valid service, deposit rule, hours, and cancellation window configured, with no partial state left behind.

**Rationale:** Every downstream Core feature (availability, deposit, no-show handling) depends on this configuration being complete; an incomplete setup silently breaks the rest of the product for that Pro. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Business Settings & Policy Configuration

---

### Personal Time Protected

**Description:** Measures whether time Mara manually blocks off is ever incorrectly shown as bookable to a client.

**Target:** Zero instances of a client being able to book a slot that falls within a manually blocked span.

**Rationale:** Manual blocking is the direct, immediate backstop for the brief's non-negotiable double-booking promise, independent of and faster than external calendar sync. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Pro Manual Schedule Blocking

---

### Client Record Reliability

**Description:** Measures whether every client who books is correctly and automatically added to Mara's private client list, with notes preserved across future bookings.

**Target:** 100% of confirmed bookings result in an accurate, findable Client Record, with previously recorded notes intact on that client's next visit.

**Rationale:** BRIEF.md states Mara "sees every booking, every client" (Target Users & Roles); a missed or duplicated client record breaks the relationship continuity the brief describes. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Client Record Management

---

### Calendar Sync Accuracy

**Description:** Measures how quickly and reliably a Pro's external calendar busy time is reflected in her availability here, and vice versa.

**Target:** A new personal calendar event blocks matching availability within a few minutes on average, and a confirmed booking appears on the Pro's personal calendar within the same window.

**Rationale:** BRIEF.md confirms two-way sync end-to-end (Ecosystem & Integrations); slow or unreliable sync directly threatens the double-booking-integrity Success Criterion. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Two-Way Calendar Sync

---

### First-Session Onboarding Completion

**Description:** Measures whether a new Pro reaches a working, shareable bio-link URL within a single first session, rather than needing to return to finish setup.

**Target:** At least 80% of new Pros complete onboarding (account, at least one service, policy, hours) and receive their bio-link URL within their first session.

**Rationale:** The founder's own three-month goal of a first paying Pro (BRIEF.md, Constraints) depends on Pros reaching a usable product quickly; an onboarding flow that requires multiple sessions to finish is a leading indicator of drop-off before the first booking ever happens. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Pro Onboarding & Setup

---

### Account Recovery Success

**Description:** Measures whether a Pro who loses access to her account can regain it without losing any of her data.

**Target:** A Pro who initiates account recovery regains full access, with all services, policies, bookings, and client records intact, without needing manual founder intervention in the ordinary case.

**Rationale:** A solo Pro with no IT support of her own cannot afford to be locked out of her own business; recovery failures directly threaten her livelihood, not just convenience. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Pro Account & Authentication

---

### Deposit Payment Reliability

**Description:** Measures whether a client's card deposit is captured successfully on a valid card, without spurious failures unrelated to the card itself.

**Target:** At least 98% of deposit payment attempts on a valid, sufficiently-funded card complete successfully on the first try.

**Rationale:** A deposit that fails to capture for reasons unrelated to the client's own card directly breaks the brief's central "pay a card deposit" mechanism and erodes trust in both roles at once. [CHALLENGED: GlossGenius reviews report card validation for deposits creates booking friction for some clients, particularly with debit cards, on an otherwise comparable flat-subscription product (source: App Store reviews, confidence: MEDIUM) -- target retained per SYN-04 protection (deposit-by-card is the brief's own non-negotiable mechanism); flagged for Stage 3/4 card-entry UX attention.]

**Persona:** All

**Connected Feature:** Payment Processing Capability

---

### Message Delivery Reliability

**Description:** Measures whether confirmation and reminder texts reliably reach clients who have consented to receive them.

**Target:** At least 98% of consent-gated confirmation and reminder messages are successfully delivered.

**Rationale:** BRIEF.md's reminder mechanism is the direct replacement for Mara's manual, late-night texting (Problem Statement); undelivered reminders silently reintroduce the exact problem the product exists to solve. [INFERRED: carried from Visionary draft]

**Persona:** All

**Connected Feature:** SMS Messaging & Consent Capability

---

### Subscription Billing Reliability

**Description:** Measures whether Mara's own flat monthly subscription charge processes correctly and predictably, without unexpected interruptions to her booking capability.

**Target:** At least 98% of monthly subscription charges succeed on the first attempt, and any failure gives Mara a grace period before her booking page is affected.

**Rationale:** BRIEF.md's stated business model is a flat monthly charge (Business Context); a billing failure that silently takes Mara's booking page offline would be more damaging to trust than the modest fee itself. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Pro Subscription & Billing

---

### Dispute Resolution Confidence

**Description:** Measures whether Mara can produce a concrete, timestamped answer the moment a client disputes a no-show or forfeited deposit.

**Target:** 100% of disputed bookings have a complete, unbroken history (policy agreed, payment timestamp, status-change timestamps) available to Mara at the moment she needs it.

**Rationale:** This metric directly answers the brief's named pain point: "no record when a client disputes a no-show charge" (Problem Statement). [RESEARCH-INFORMED: every profiled competitor with reviewable enforcement evidence has at least one documented case of deposit/no-show enforcement failing silently or without notification (source: market-research.md, Common Complaint Themes, confidence: HIGH); this metric targets exactly the reliability gap the market has not solved, not only the brief's own pain point.]

**Persona:** Mara

**Connected Feature:** Booking Record & Dispute Trail

---

### Client Self-Service Awareness

**Description:** Measures whether clients actually find and use their own booking history view when they want to check an upcoming appointment, rather than messaging Mara to ask.

**Target:** At least 75% of clients with an upcoming booking check it through their own booking history view rather than messaging Mara directly to confirm details.

**Rationale:** If clients still message Mara to ask "when's my appointment again," the self-service view isn't reducing her DM burden the way the brief envisions. [INFERRED: carried from Visionary draft]

**Persona:** Taylor

**Connected Feature:** Client Self-Service Booking History

---

### Clean Account Closure

**Description:** Measures whether a Pro who closes her account, or deletes a client's record on request, ends up with a clean, unambiguous result — nothing orphaned, nothing half-deleted.

**Target:** 100% of account closures and client-record deletions leave no orphaned bookings, no continued billing, and no ambiguity about what was deleted versus retained.

**Rationale:** BRIEF.md's privacy constraint requires that "a pro must be able to delete a client's record on request" (Constraints) cleanly, not partially — a half-completed deletion is functionally a broken promise. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Account Closure & Client Data Deletion

---

### Support Diagnosis Speed

**Description:** Measures whether the founder, acting as Operator, can see enough of a reported issue's context to understand what happened without needing to ask the Pro follow-up questions first.

**Target:** The Operator can identify the relevant booking, its status history, and the Pro's active policy for a reported issue within the read-only console, without a back-and-forth clarifying exchange with the Pro in the ordinary case.

**Rationale:** BRIEF.md confirms this support capability exists specifically so the founder can help without acting on the Pro's behalf (Target Users & Roles); if the console doesn't surface enough context, it fails its one job. [RESEARCH-INFORMED: support responsiveness is a documented, cross-competitor weak point — slow response, multi-agent hand-offs, and chatbot-first support drew complaints across Booksy, theCut, and Fresha (source: market-research.md, Common Complaint Themes, confidence: HIGH) — reinforcing why a founder who can self-diagnose quickly, without relying on a support queue, is a deliberate advantage worth measuring.]

**Persona:** All

**Connected Feature:** Operator Support Console

---

### No-DM Adoption

**Description:** Measures whether Pros actually stop taking bookings by Instagram DM once they adopt the product, rather than running both in parallel indefinitely.

**Target:** At least 80% of active Pros report taking zero new bookings by DM within 60 days of going live with their bio link.

**Rationale:** This is a headline brief Success Criterion in the founder's own words: "pros stop taking bookings by DM entirely — the link is the only way to book them" (Success Criteria). [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Public Booking Page

---

### Week-8 Pro Retention

**Description:** Measures whether Pros who go live with a working bio link are still active subscribers two months later, the clearest signal the product has replaced their old workflow rather than being tried and abandoned.

**Target:** At least 70% of Pros who complete onboarding and take at least one real booking are still active subscribers at week 8.

**Rationale:** The brief's business model depends on Pros staying subscribed because the product is genuinely better than DMs and a paper diary, not on one-time signups; early churn would indicate the core loop isn't delivering the promised relief. [INFERRED: carried from Visionary draft]

**Persona:** Mara

**Connected Feature:** Pro Subscription & Billing

---

### Peer Referral Growth

**Description:** Measures what share of new Pros arrive because another Pro told them about the product, rather than through paid acquisition.

**Target:** At least half of new Pro signups in a given month cite or can be attributed to a referral from an existing Pro.

**Rationale:** This is a headline brief Success Criterion: "most new pros arrive because another pro told them about it" (Success Criteria) — a direct measure of whether the product earns organic trust within this close-knit professional community. [RESEARCH-INFORMED: market research confirms solo/independent-first positioning is an active, recognized market segment with its own word-of-mouth dynamics — GlossGenius and theCut both validate purpose-built solo-operator products as a distinct category from broader salon-suite software (source: market-research.md, GlossGenius and theCut profiles, confidence: MEDIUM), consistent with peer referral being a credible primary growth channel rather than an optimistic assumption.]

**Persona:** Mara

**Connected Feature:** Pro Onboarding & Setup

---

### Client Repeat Booking Rate

**Description:** Measures whether clients who book once with a Pro come back through the link again for their next appointment, rather than reverting to DM-ing the Pro directly the second time around.

**Target:** At least 60% of clients with more than one appointment in a 6-month window booked their most recent appointment through the link rather than through a message to the Pro.

**Rationale:** The brief's success only holds if the link becomes "the only way to book," not just the way a client books once out of novelty; repeat-booking behavior is the clearest sign the habit has actually replaced DM-ing. [INFERRED: carried from Visionary draft]

**Persona:** Taylor

**Connected Feature:** Client Record Management

---

### Availability Responsiveness

**Description:** Measures whether the live availability view feels instant to a client picking a time, since this is the step most exposed to the flaky connectivity of Instagram's in-app browser.

**Target:** Free slots render within about 1 second of selecting a service, even on a typical mobile connection inside Instagram's in-app browser.

**Rationale:** BRIEF.md specifies the product "must work well" inside Instagram's in-app browser (Scale & Non-Functional Expectations); a slow-feeling slot picker is the most likely point where an impatient client abandons the booking. [INFERRED: carried from Visionary draft]

**Persona:** Taylor

**Connected Feature:** Live Availability & Slot Booking
</content>
