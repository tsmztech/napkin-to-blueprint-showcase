---
document_type: user-journeys
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# User Journeys

## Journey

This document contains 7 journeys covering the full lifecycle for both product roles: two first-use journeys (Pro setup, Client's first booking), three regular-use journeys (the Pro's daily rhythm, a client managing an existing booking, and the Pro handling cancellations/reschedules), and two edge/recovery journeys (a no-show dispute, and a declined payment). Talia (the Pro) owns four journeys, Riley (the Client) owns three. Every Core and Important feature in draft-product-features.md appears in at least one journey.

---

### Pro First-Time Setup

**Owning Persona:** Talia

**Coverage:** First-use

**Journey Goal:** Get from signing up to a live, shareable booking link that Talia can put in her Instagram bio, with her real rules already in place.

**Entry Point:** Talia signs up for Chairtime after a friend (another pro) tells her about it, motivated to stop negotiating times in DMs.

**Steps:**

1. Account start — Talia begins the guided setup. She sees a short, plain-language explanation of what each step will ask for, with no jargon.
2. Services and deposit rule — Talia adds her first service (name, price, duration) and sets her deposit rule (fixed amount or percentage). She sees exactly how a client will see this service on her future booking page.
3. Hours and cancellation policy — Talia sets her working hours and buffer time, then sets her cancellation window and what happens to the deposit inside vs. outside it, choosing the recommended default or adjusting it.
4. Calendar connection — Talia connects her personal Google or Apple calendar so her real busy time blocks Chairtime automatically; she is told she can skip this and connect later.
5. Subscription — Talia enters her card for the flat monthly subscription. Her account is now active.
6. Link is live — Talia is handed her shareable booking link and a short "here's what to do with it" note (drop it in her Instagram bio).

**Failure/Recovery Variant:** Talia closes the app partway through, right after setting her services, to attend to a client. When she reopens Chairtime later that day, setup resumes exactly at the hours/cancellation-policy step with her services already saved — nothing is lost and she is never asked to start over.

**Success Outcome:** Within one sitting (or across a couple of short sessions), Talia has a live booking link reflecting her real services, hours, deposit rule, and cancellation policy, with her calendar connected and her subscription active.

**Connected Features:** Pro Onboarding & Setup Wizard, Service & Pricing Management, Availability & Working Hours Setup, Cancellation & No-Show Policy Engine, Two-Way Calendar Sync, Pro Subscription Billing & Account Management.

---

### Client's First Booking

**Owning Persona:** Riley

**Coverage:** First-use

**Journey Goal:** Book an appointment with a pro discovered on Instagram, and pay the deposit, in under a minute, without creating an account.

**Entry Point:** Riley sees a fresh set of lashes on the pro's Instagram feed and taps the link in the bio.

**Steps:**

1. Land on the booking page — Riley sees the pro's name, services with prices and durations, and the deposit rule stated in plain words, right inside the Instagram in-app browser.
2. Pick a service and time — Riley chooses "Full set — $65 — 90 min" and sees only times that are genuinely free; they pick Thursday 2:30pm.
3. Enter details and consent — Riley enters their name and phone number and actively opts in to receive text messages.
4. Pay the deposit — Riley pays the $20 deposit by card. The page confirms success immediately.
5. Confirmation lands — Riley sees an on-screen confirmation, and a confirmation text arrives moments later.

**Failure/Recovery Variant:** Riley's card is declined at the payment step. The page shows the decline reason in plain language, keeps Thursday 2:30pm held for a short window, and lets Riley retry with a different card without re-entering their name, phone, or service choice.

**Success Outcome:** In under a minute, Riley has a confirmed, paid booking and a confirmation text, without ever creating a password or account.

**Connected Features:** Public Booking Page & Booking Flow, Real-Time Slot Availability Engine, Client Booking Identity, Deposit Payment at Booking, Automated Booking Messaging, Messaging Consent Management, Cancellation & No-Show Policy Engine.

---

### Talia's Between-Clients Day

**Owning Persona:** Talia

**Coverage:** Regular

**Journey Goal:** Glance at today's schedule between clients, confirm who's paid and who's next, and handle a no-show without any manual chasing.

**Entry Point:** Talia has a few minutes between appointments and opens Chairtime on her phone, as she does throughout most working days.

**Steps:**

1. Open the dashboard — Talia sees today's remaining bookings in time order, each with a paid badge and balance-due amount.
2. Check a client note — Talia taps into her next booking and sees a private note she left last time ("prefers a lighter volume").
3. Handle a blocked afternoon — Talia adds a manual time block for a personal appointment later in the week directly from the schedule view.
4. Mark a no-show — Her 11am client never arrives. Talia taps "no-show" on that booking; the deposit is automatically kept under her policy, with nothing further for her to do.

**Failure/Recovery Variant:** Talia realizes moments later that she marked the wrong booking as a no-show (her actual 11am client texted that they were running five minutes late and did arrive). She undoes the no-show mark within the grace period, and the booking and deposit status both revert cleanly.

**Success Outcome:** Talia has moved through her day with a clear, trustworthy view of her schedule and handled a genuine no-show without a single manual invoice, text, or negotiation.

**Connected Features:** Pro Daily Schedule Dashboard, No-Show Marking & Deposit Forfeiture, Client Record Management, Manual Time Blocking.

---

### Riley Manages an Existing Booking

**Owning Persona:** Riley

**Coverage:** Regular

**Journey Goal:** Respond to a reminder and reschedule an upcoming appointment without calling or messaging the pro directly.

**Entry Point:** Riley receives the automatic reminder text two days before their appointment.

**Steps:**

1. Reminder arrives — Riley reads the reminder, which includes one-tap "I'll be there" and "I need to reschedule" options.
2. Choose to reschedule — Riley taps "I need to reschedule" and is taken straight into their own booking, verified by the phone number the reminder was sent to.
3. Pick a new time — Riley sees the same real-time list of genuinely free slots and picks a new one for the same service.
4. Confirmation — The booking updates to the new time; Riley gets a confirmation, and the pro's calendar reflects the change automatically.

**Failure/Recovery Variant:** Riley's first-choice new time disappears from the list moments after they open it (someone else just booked it). The page shows a plain "that time was just taken" message and keeps Riley on the same live slot list to pick another, rather than erroring out.

**Success Outcome:** Riley moves their appointment to a time that works, in a couple of taps, with no phone call and no re-paying a deposit.

**Connected Features:** Automated Booking Messaging, Client Booking Identity, Client-Initiated Cancel/Reschedule, Real-Time Slot Availability Engine.

---

### Talia Handles Cancellations and Freed-Up Time

**Owning Persona:** Talia

**Coverage:** Regular

**Journey Goal:** See a client's cancellation reflected correctly (deposit refunded or kept, per policy) and let a waitlisted client claim the newly freed slot.

**Entry Point:** A client cancels their own booking from a reminder text, well inside Talia's cancellation window.

**Steps:**

1. Cancellation lands — Talia's dashboard shows the booking as cancelled, with the deposit automatically kept because the cancellation fell inside her stated window.
2. Slot reopens — The freed time immediately becomes bookable again on Talia's public page.
3. Waitlist notified — A different client who had joined the waitlist for that day is notified that a matching slot just opened.
4. New booking lands — That client books the freed slot within the priority window; Talia's schedule fills back in without her doing anything.

**Failure/Recovery Variant:** No one on the waitlist claims the slot before the priority window expires. The slot simply returns to ordinary public availability, and Talia's schedule shows it as open — nothing is lost or stuck in a pending state.

**Success Outcome:** A cancellation resolves itself correctly (deposit outcome, calendar update, and re-booking opportunity) with no manual intervention from Talia.

**Connected Features:** Client-Initiated Cancel/Reschedule, Cancellation & No-Show Policy Engine, Waitlist for Cancelled Slots, Automated Booking Messaging.

---

### Resolving a No-Show Dispute

**Owning Persona:** Talia

**Coverage:** Edge/Recovery

**Journey Goal:** Respond confidently to a client who disputes a kept deposit, using a trustworthy record instead of memory or guesswork.

**Entry Point:** A client messages Talia directly (outside the app) disputing why their deposit was kept for a missed appointment.

**Steps:**

1. Talia opens the booking — She finds the booking in question from her dashboard and opens its full activity timeline.
2. Timeline review — She sees exactly when the client booked, which cancellation policy version was shown to them at that moment, the appointment time, and the timestamp she marked it as a no-show.
3. Talia responds — Using the timeline as reference, Talia explains the outcome to the client with confidence, pointing to the exact policy they agreed to.
4. Escalation (if needed) — If Talia is unsure how to interpret something, she contacts support; a support operator opens the same read-only timeline to help, without ever seeing Talia's private notes about the client.

**Failure/Recovery Variant:** The client insists the timeline must be wrong. Because every entry is append-only and immutable, Talia can point to the exact, unaltered sequence of events rather than an editable log that could be doubted — the record itself is the recovery mechanism for the dispute.

**Success Outcome:** Talia resolves the dispute in minutes with a clear, trustworthy record, rather than losing an evening to an argument she cannot substantiate.

**Connected Features:** No-Show Marking & Deposit Forfeiture, Booking & Payment Activity Record, Platform Support Read-Only Access, Cancellation & No-Show Policy Engine.

---

### Recovering from a Declined Deposit Payment

**Owning Persona:** Riley

**Coverage:** Edge/Recovery

**Journey Goal:** Successfully complete a booking after an initial payment attempt fails, without losing the chosen time or having to start over.

**Entry Point:** Riley reaches the payment step of booking and their card is declined.

**Steps:**

1. Decline shown — Riley sees a clear, specific reason for the decline rather than a generic error.
2. Slot still held — Riley's chosen Thursday 2:30pm slot remains held for a short window rather than being released back to the public list immediately.
3. Retry with a different card — Riley re-enters payment details with a different card, without re-selecting the service, time, name, or phone.
4. Success — The deposit succeeds on the second attempt, and Riley gets the same confirmation experience as a first-attempt success.

**Failure/Recovery Variant:** Riley takes too long deciding what card to use and the slot hold expires before they retry. Riley sees a plain "that hold has expired, please pick a time again" message and is returned to the live slot list — never charged, and never left uncertain about whether they are booked.

**Success Outcome:** Riley completes their booking despite an initial payment hiccup, with no confusion about whether they were charged or whether their slot was lost.

**Connected Features:** Deposit Payment at Booking, Public Booking Page & Booking Flow, Real-Time Slot Availability Engine.
