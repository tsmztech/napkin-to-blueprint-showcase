---
document_type: success-metrics
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed (1 fix applied)
---

# Success Metrics

## Summary

This document contains 20 success metrics, covering all 14 Core features and 2 Important features: 13 product-experience metrics, 4 adoption/engagement/business KPIs, and 3 user-facing performance expectations. Together they validate the founder's two headline promises — "client books in a minute" and "pro never chases a no-show again" — plus the reliability and business-viability bar the brief sets alongside them. [MODIFIED: four metrics added -- Availability Setup Accuracy for Availability & Working Hours Setup, which the draft claimed was covered but had no metric (synthesis check fix), and Automatic Refund Correctness, Payout Transparency and Pro Change Correctness for the refund path and the two audit-added Core features]

---

### Booking Completion Speed

**Description:** Measures how quickly a client can go from opening the pro's booking link to a confirmed, paid appointment.

**Target:** The client can complete a booking — from opening the link to seeing an on-screen confirmation — in under one minute, matching BRIEF.md's own stated benchmark.

**Rationale:** BRIEF.md's Vision states this exact benchmark: "Client books in a minute." This is the product's single clearest experience promise.

**Persona:** Riley

**Connected Feature:** Public Booking Page & Booking Flow

---

### Deposit Capture Rate

**Description:** Measures how reliably a chosen deposit rule results in a successfully captured payment at booking, without silent failures or ambiguous states.

**Target:** Every booking attempt that reaches the payment step (100%) ends in either a clear success or a clear, actionable decline — never an ambiguous or lost state — and at least 90% of attempts that reach the payment step end in a paid, confirmed booking. [MODIFIED: the ambiguity target raised from 95% to 100% because BRIEF.md's Success Criteria treat a lost deposit as an absolute ("nobody has ever had... a lost deposit"), and a separate completion share added so the metric still tracks capture]

**Rationale:** BRIEF.md's Success Criteria demands the product "never... lose a deposit." A high, clean capture (or clean-fail) rate is the direct evidence that the deposit mechanic is trustworthy.

**Persona:** Riley

**Connected Feature:** Deposit Payment at Booking

---

### Zero Double-Booking Confidence

**Description:** Measures whether the availability engine ever offers a slot that turns out not to be genuinely free.

**Target:** Zero confirmed double-bookings across all pros, ever — matching BRIEF.md's Success Criteria: "Nobody has ever had a double booking."

**Rationale:** This is stated as an absolute in BRIEF.md's Success Criteria, not a percentage target. A single double-booking is treated as a product failure serious enough to threaten the founder's word-of-mouth growth model.

**Persona:** All

**Connected Feature:** Real-Time Slot Availability Engine

---

### Calendar Sync Reliability

**Description:** Measures whether a pro's connected personal calendar reliably reflects Chairtime bookings and reliably blocks Chairtime availability from external busy time.

**Target:** A newly confirmed booking appears on the pro's connected personal calendar, and a new personal-calendar event blocks Chairtime availability, within a couple of minutes in each direction, essentially every time.

**Rationale:** BRIEF.md's Ecosystem & Integrations calls this two-way sync out as something that "both matter" for Google and Apple; a pro who cannot trust it will keep a second, manual system out of habit, defeating the product's purpose.

**Persona:** Talia

**Connected Feature:** Two-Way Calendar Sync

---

### Self-Service Access Success

**Description:** Measures whether a returning client can reliably access and manage their own booking using only their phone number, without contacting the pro.

**Target:** At least 90% of clients who request an access link successfully view or act on their booking without needing to message the pro directly.

**Rationale:** This is the mechanism that fulfills BRIEF.md's requirement that clients "must not face a signup wall or need a password-style account" while still letting them self-serve.

**Persona:** Riley

**Connected Feature:** Client Booking Identity

---

### Reminder Response Rate

**Description:** Measures how often clients engage with the automatic pre-appointment reminder (tapping "I'll be there" or "I need to reschedule") rather than ignoring it.

**Target:** At least 70% of reminders receive an explicit one-tap response before the appointment.

**Rationale:** BRIEF.md's Vision describes this exact interaction as the mechanism that replaces the pro's habit of texting reminders by hand; a high response rate shows clients are actually using it rather than the reminder becoming background noise.

**Persona:** Riley

**Connected Feature:** Automated Booking Messaging

---

### Policy Clarity at Booking

**Description:** Measures whether clients understand the cancellation/deposit policy before they agree to it, evidenced by a low rate of clients disputing an outcome they were shown at booking time.

**Target:** Fewer than 5% of forfeited deposits are followed by a client dispute claiming they didn't understand the policy.

**Rationale:** BRIEF.md's Business Context frames the entire deposit-forfeiture mechanism as something "the client agreed to when booking" — this metric validates that the plain-language presentation actually achieves informed agreement, not just technical consent. [RESEARCH-INFORMED: disputes over deposit and cancellation charges whose terms were unclear at booking are a recurring complaint in this market, making the disclosure moment the documented point where client trust is lost, from BBB complaint records and forum-derived summaries (MEDIUM confidence)]

**Persona:** Riley

**Connected Feature:** Cancellation & No-Show Policy Engine

---

### Self-Service Reschedule Rate

**Description:** Measures how often clients successfully cancel or reschedule their own booking without contacting the pro directly by phone or DM.

**Target:** At least 85% of client-initiated cancellations or reschedules are completed entirely in-app, with no direct message to the pro.

**Rationale:** BRIEF.md's Success Criteria states pros should "stop taking bookings by DM entirely." This metric extends that promise to cancellations and reschedules, not just new bookings.

**Persona:** Riley

**Connected Feature:** Client-Initiated Cancel/Reschedule

---

### No-Show Recovery Rate

**Description:** Measures the share of no-show appointments whose deposit is successfully and automatically forfeited to the pro, with no manual chasing required.

**Target:** 100% of bookings marked no-show result in the deposit correctly reflecting as kept, with zero instances requiring the pro to manually invoice or negotiate for it afterward.

**Rationale:** This is the founder's headline promise verbatim: "the pro never chases a no-show again," and BRIEF.md's Success Criteria states the specific target sentiment: "I haven't had an unpaid no-show since I switched."

**Persona:** Talia

**Connected Feature:** No-Show Marking & Deposit Forfeiture

---

### Daily Dashboard Glance Speed

**Description:** Measures whether the pro can understand their remaining day (who's next, who's paid, what's owed) in a single quick glance, matching how the brief describes real usage.

**Target:** A pro can identify their next booking's status (paid/unpaid, balance due) within a few seconds of opening the dashboard, with no extra navigation required.

**Rationale:** BRIEF.md's Vision describes this exact behavior: "As the pro, you glance at your phone between clients." The dashboard is the pro's single most frequent touchpoint, so its speed defines the perceived speed of the whole product for her.

**Persona:** Talia

**Connected Feature:** Pro Daily Schedule Dashboard

---

### Setup-to-Live-Link Completion

**Description:** Measures whether a new pro can get from signing up to a live, shareable booking link without needing help.

**Target:** At least 80% of new pros who begin onboarding reach a live booking link within their first session, without contacting support.

**Rationale:** BRIEF.md's Constraints state the founder's own goal of "the first paying pro within about three months" — onboarding friction is a direct threat to that timeline, and a pro who cannot self-serve setup is a pro who may not return.

**Persona:** Talia

**Connected Feature:** Pro Onboarding & Setup Wizard

---

### Service Setup Confidence

**Description:** Measures whether a pro can add a service, price, duration, and deposit rule correctly on the first attempt, without confusing errors or unclear validation.

**Target:** A pro can add a new service and see it correctly reflected on their public booking page within a couple of minutes, with no failed or abandoned attempts due to unclear validation.

**Rationale:** Service setup is the first substantive configuration a pro performs, and it directly determines what a client sees during their under-one-minute booking (Booking Completion Speed, above) — errors here propagate straight to the client experience.

**Persona:** Talia

**Connected Feature:** Service & Pricing Management

---

### DM-to-Link Migration

**Description:** Measures whether pros stop taking bookings through Instagram DMs entirely once they adopt Chairtime, using only the booking link going forward.

**Target:** Within one month of onboarding, at least 90% of a pro's new bookings arrive through the Chairtime link rather than being negotiated in DMs and entered manually.

**Rationale:** BRIEF.md's Success Criteria states this as a defining outcome: "Pros stop taking bookings by DM entirely — the link is the only way to book them." This is the clearest behavioral signal that the product has actually replaced the old habit, not just supplemented it.

**Persona:** Talia

**Connected Feature:** Public Booking Page & Booking Flow

---

### Peer-Referral Growth Share

**Description:** Measures what share of new pros join because another pro told them about Chairtime, rather than through paid acquisition.

**Target:** A majority of new pro sign-ups in year one cite another pro as the reason they found Chairtime.

**Rationale:** BRIEF.md's Success Criteria and Business Context both name this as the intended growth engine: "most new pros arrive because another pro told them about it," and "go-to-market is the founder's own network of pros plus peer referral."

**Persona:** All

**Connected Feature:** Pro Onboarding & Setup Wizard

---

### Subscription Retention

**Description:** Measures whether pros continue their flat monthly subscription month over month rather than churning after a short trial period.

**Target:** At least 80% of pros who complete their first paid month remain subscribed into their fourth month.

**Rationale:** BRIEF.md's Business Context ties the entire pricing model to sustained value: "priced so that a single saved no-show pays for the month." Sustained retention is the evidence that this value proposition is actually being felt, not just promised at signup. [RESEARCH-INFORMED: fee unpredictability, not the headline price, is the dominant trust complaint across three competitors (HIGH confidence), so retention also tests whether a single flat, all-inclusive price holds trust over time]

**Persona:** Talia

**Connected Feature:** Pro Subscription Billing & Account Management

---

### Availability Setup Accuracy

[MODIFIED: added because the draft summary claimed every Core feature had a metric but Availability & Working Hours Setup had none -- synthesis check fix]

**Description:** Measures whether the times a client is offered match what the pro actually intended when setting hours, buffers, and notice limits.

**Target:** A pro who opens their own booking page after setting their hours sees exactly the times they expected, and fewer than 1 in 50 pros need to contact support or re-edit their hours because offered times did not match their intent in their first month.

**Rationale:** BRIEF.md's Target Users & Roles has the pro set "working hours, buffer time between clients" once and then trust it; if offered times surprise the pro, they will go back to negotiating in DMs, defeating the "genuinely free time" promise in BRIEF.md's Vision.

**Persona:** Talia

**Connected Feature:** Availability & Working Hours Setup

---

### Automatic Refund Correctness

[AUDIT-ADDED: 1 -- value-flow walk: the refund exit path for deposits had no success measure]

**Description:** Measures whether every deposit that the pro's policy says should be refunded actually reaches the client, without the pro lifting a finger.

**Target:** 100% of cancellations made outside the pro's window, and 100% of cancellations made by the pro, result in the client seeing their full deposit refund confirmed, and every refund that cannot complete immediately is shown to both parties as "in progress" and completes without either person having to chase it.

**Rationale:** BRIEF.md's Business Context states that "a cancellation outside the window refunds the deposit automatically," and its Success Criteria allow no lost deposits; a refund that silently fails is a lost deposit from the client's side.

**Persona:** All

**Connected Feature:** Cancellation & No-Show Policy Engine

---

### Payout Transparency

[AUDIT-ADDED: 1 -- the audit-added Core feature Payout Account Connection & Payout Visibility needed a metric]

**Description:** Measures whether a pro can see where their deposit money is and understand every deduction, without asking.

**Target:** A pro can find any deposit, refund, processor fee, or bank payout from the last 90 days in their money list within a few seconds, and no pro ever sees a deduction labeled as a Chairtime fee.

**Rationale:** BRIEF.md's Business Context promises "the platform takes no cut of any of it," and market research finds unpredictable, layered fees to be the dominant trust complaint in this market (HIGH confidence); visible, explained money flow is how the promise is kept observably.

**Persona:** Talia

**Connected Feature:** Payout Account Connection & Payout Visibility

---

### Pro Change Correctness

[AUDIT-ADDED: 1 -- the audit-added Core feature Pro Booking Management needed a metric]

**Description:** Measures whether a pro can cancel, reschedule, or rebook clients themselves quickly, with every client notified and every deposit handled correctly.

**Target:** A pro can cancel or reschedule a booking in under 30 seconds from the dashboard; 100% of pro-initiated cancellations refund the client's deposit in full and notify the client; and at least 70% of deposit requests the pro sends when rebooking at the chair are paid before the hold expires.

**Rationale:** BRIEF.md's Target Users & Roles lets the pro "reschedule, cancel and refund within policy," and its Success Criteria say "the link is the only way to book them"; a fast, fair pro-side path keeps pros from falling back to DMs and hand-sent refunds.

**Persona:** Talia

**Connected Feature:** Pro Booking Management

---

### Slot Search Responsiveness

**Description:** Measures how quickly the client-facing slot list appears and updates, since this is inside the product's most frequent and time-sensitive loop.

**Target:** Available slots for a chosen service appear within roughly one second of selection, and the list updates within roughly one second of a slot being taken by another client.

**Rationale:** The under-one-minute booking promise (Booking Completion Speed, above) depends on availability never becoming a visible wait. This is a user-facing performance expectation — the client experiences "instant" or "sluggish" directly — aligned with the Non-Functional Expectations section of draft-assumptions-constraints.md.

**Persona:** Riley

**Connected Feature:** Real-Time Slot Availability Engine
