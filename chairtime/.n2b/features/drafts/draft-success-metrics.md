---
document_type: success-metrics
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# Success Metrics

## Summary

This document contains 16 success metrics, covering all 12 Core features and 2 Important features: 9 product-experience metrics, 4 adoption/engagement/business KPIs, and 3 user-facing performance expectations. Together they validate the founder's two headline promises — "client books in a minute" and "pro never chases a no-show again" — plus the reliability and business-viability bar the brief sets alongside them.

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

**Target:** At least 95% of booking attempts that reach the payment step end in either a clear success or a clear, actionable decline — never an ambiguous or lost state.

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

**Rationale:** BRIEF.md's Business Context frames the entire deposit-forfeiture mechanism as something "the client agreed to when booking" — this metric validates that the plain-language presentation actually achieves informed agreement, not just technical consent.

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

**Rationale:** BRIEF.md's Business Context ties the entire pricing model to sustained value: "priced so that a single saved no-show pays for the month." Sustained retention is the evidence that this value proposition is actually being felt, not just promised at signup.

**Persona:** Talia

**Connected Feature:** Pro Subscription Billing & Account Management

---

### Slot Search Responsiveness

**Description:** Measures how quickly the client-facing slot list appears and updates, since this is inside the product's most frequent and time-sensitive loop.

**Target:** Available slots for a chosen service appear within roughly one second of selection, and the list updates within roughly one second of a slot being taken by another client.

**Rationale:** The under-one-minute booking promise (Booking Completion Speed, above) depends on availability never becoming a visible wait. This is a user-facing performance expectation — the client experiences "instant" or "sluggish" directly — aligned with the Non-Functional Expectations section of draft-assumptions-constraints.md.

**Persona:** Riley

**Connected Feature:** Real-Time Slot Availability Engine
