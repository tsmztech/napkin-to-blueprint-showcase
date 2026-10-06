---
document_type: user-journeys
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed (1 fix applied)
---

# User Journeys

## Journey

This document contains 8 journeys covering the full lifecycle for both product roles: two first-use journeys (Pro setup, Client's first booking), three regular-use journeys (the Pro's daily rhythm, a client managing an existing booking, and the Pro handling cancellations and freed-up time), and three edge/recovery journeys (a no-show dispute, a declined payment, and the Pro cancelling a sick day). Talia (the Pro) owns five journeys, Riley (the Client) owns three. Every Core and Important feature in product-features.md appears in at least one journey. [MODIFIED: journey count raised from 7 to 8 so the set meets the one-journey-per-four-features coverage rule for the final 30-feature set, adding "Talia Cancels a Sick Day" to cover the audit-added Pro Booking Management and Payout Account Connection & Payout Visibility features]

---

### Pro First-Time Setup

**Owning Persona:** Talia

**Coverage:** First-use

**Journey Goal:** Get from signing up to a live, shareable booking link that Talia can put in her Instagram bio, with her real rules already in place and her deposits set up to reach her.

**Entry Point:** Talia signs up for Chairtime after a friend (another pro) tells her about it, motivated to stop negotiating times in DMs.

**Steps:**

1. Account and profile — Talia creates her sign-in with her email and mobile number and a one-time code (no password to remember), then adds her display name, a photo, and her studio location. She sees a short, plain-language explanation of what each step will ask for, with no jargon. [MODIFIED: sign-in and profile added to the first step because the completeness audit found setup otherwise left the Pro with no way to sign back in and clients with no studio location]
2. Services and deposit rule — Talia adds her first service (name, price, duration) and sets her deposit rule (fixed amount or percentage). She sees exactly how a client will see this service on her future booking page.
3. Hours and cancellation policy — Talia sets her working hours and buffer time, then sets her cancellation window and what happens to the deposit inside vs. outside it, choosing the recommended default or adjusting it.
4. Getting paid and calendar — Talia connects her payout account through the payment processor's own verification, so client deposits land directly with her, and sees plainly that Chairtime takes nothing from them. She then connects her personal Google or Apple calendar so her real busy time blocks Chairtime automatically; she is told she can skip the calendar and connect it later. [MODIFIED: payout-account connection added because the value-flow audit found deposits had nowhere to go]
5. Subscription — Talia enters her card for the flat monthly subscription, shown as one all-inclusive price. Her account is now active.
6. Link is live — Talia previews her booking page exactly as a client will see it, then is handed her shareable booking link and a short "here's what to do with it" note (drop it in her Instagram bio).

**Failure/Recovery Variant:** Talia closes the app partway through, right after setting her services, to attend to a client. When she reopens Chairtime later that day, setup resumes exactly at the hours/cancellation-policy step with her services already saved — nothing is lost and she is never asked to start over. If her payout verification is still pending at the end, everything else is kept and her link simply waits, with a clear "finish verifying to start taking bookings" prompt.

**Success Outcome:** Within one sitting (or across a couple of short sessions), Talia has a live booking link reflecting her real services, hours, deposit rule, and cancellation policy, with her payout account active, her calendar connected, and her subscription active.

**Connected Features:** Pro Onboarding & Setup Wizard, Pro Sign-In & Account Lifecycle, Pro Profile & Booking Page Settings, Service & Pricing Management, Availability & Working Hours Setup, Cancellation & No-Show Policy Engine, Payout Account Connection & Payout Visibility, Two-Way Calendar Sync, Pro Subscription Billing & Account Management.

---

### Client's First Booking

**Owning Persona:** Riley

**Coverage:** First-use

**Journey Goal:** Book an appointment with a pro discovered on Instagram, and pay the deposit, in under a minute, without creating an account.

**Entry Point:** Riley sees a fresh set of lashes on the pro's Instagram feed and taps the link in the bio.

**Steps:**

1. Land on the booking page — Riley sees the pro's name, photo, services with prices and durations, and the deposit rule stated in plain words, right inside the Instagram in-app browser.
2. Pick a service and time — Riley chooses "Full set — $65 — 90 min" and sees only times that are genuinely free, labeled in the pro's timezone; they pick Thursday 2:30pm.
3. Enter details and consent — Riley enters their name and phone number and actively opts in to receive text messages.
4. Agree and pay — Riley sees this booking's policy in plain words ("$20 deposit; cancel before Wednesday 2:30pm for a full refund, after that it's kept"), ticks to agree, and pays the $20 deposit by card. The page confirms success immediately. [MODIFIED: explicit policy acknowledgment added before payment, based on market evidence that clients dispute deposit charges when terms were unclear at booking (2 source types, MEDIUM confidence)]
5. Confirmation lands — Riley sees an on-screen confirmation, and a confirmation text arrives moments later with the studio address, the balance due in person, the cancellation cut-off, a manage link, and an "add to my calendar" option.

**Failure/Recovery Variant:** Riley's card is declined at the payment step. The page shows the decline reason in plain language, keeps Thursday 2:30pm held for a short window, and lets Riley retry with a different card without re-entering their name, phone, or service choice.

**Success Outcome:** In under a minute, Riley has a confirmed, paid booking and a confirmation text telling them where to go and when the cancellation window closes, without ever creating a password or account.

**Connected Features:** Public Booking Page & Booking Flow, Real-Time Slot Availability Engine, Client Booking Identity, Deposit Payment at Booking, Automated Booking Messaging, Messaging Consent Management, Cancellation & No-Show Policy Engine, Pro Profile & Booking Page Settings.

---

### Talia's Between-Clients Day

**Owning Persona:** Talia

**Coverage:** Regular

**Journey Goal:** Glance at today's schedule between clients, confirm who's paid and who's next, rebook regulars, and handle a no-show without any manual chasing.

**Entry Point:** Talia has a few minutes between appointments and opens Chairtime on her phone, as she does throughout most working days.

**Steps:**

1. Open the dashboard — Talia sees today's remaining bookings in time order, each with a paid badge, balance-due amount, and whether the client tapped "I'll be there."
2. Check a client note — Talia taps into her next booking and sees a private note she left last time ("prefers a lighter volume").
3. Finish and rebook — After her 10am client, Talia marks the appointment completed (balance paid in person) and books the client's next fill three weeks out; the client scans a deposit link from Talia's screen and pays it before leaving. [AUDIT-ADDED: 1 -- journey walk: the draft had no step where a booking was completed or where a regular was rebooked at the chair]
4. Handle a blocked afternoon — Talia adds a manual time block for a personal appointment later in the week directly from the schedule view.
5. Mark a no-show — Her 11am client never arrives. Talia taps "no-show" on that booking; the deposit is automatically kept under her policy, with nothing further for her to do.

**Failure/Recovery Variant:** Talia realizes moments later that she marked the wrong booking as a no-show (her actual 11am client texted that they were running five minutes late and did arrive). She undoes the no-show mark within the 24-hour grace period, and the booking and deposit status both revert cleanly.

**Success Outcome:** Talia has moved through her day with a clear, trustworthy view of her schedule, rebooked a regular without a single DM, and handled a genuine no-show without a single manual invoice, text, or negotiation.

**Connected Features:** Pro Daily Schedule Dashboard, No-Show Marking & Deposit Forfeiture, Pro Booking Management, Client Record Management, Manual Time Blocking, Deposit Payment at Booking.

---

### Riley Manages an Existing Booking

**Owning Persona:** Riley

**Coverage:** Regular

**Journey Goal:** Respond to a reminder and reschedule an upcoming appointment without calling or messaging the pro directly.

**Entry Point:** Riley receives the automatic reminder text two days before their appointment.

**Steps:**

1. Reminder arrives — Riley reads the reminder, which includes one-tap "I'll be there" and "I need to reschedule" options.
2. Choose to reschedule — Riley taps "I need to reschedule" and is taken straight into that booking through the booking-specific manage link in the reminder, with no code or password. [MODIFIED: access now described as the booking-specific manage link carried in the reminder, which the audit added to Client Booking Identity so the tap lands directly in the booking]
3. Pick a new time — Riley sees the same real-time list of genuinely free slots and picks a new one for the same service; because this is outside the pro's cancellation window, the deposit carries over.
4. Confirmation — The booking updates to the new time; Riley gets a confirmation, and the pro's calendar reflects the change automatically.

**Failure/Recovery Variant:** Riley's first-choice new time disappears from the list moments after they open it (someone else just booked it). The page shows a plain "that time was just taken" message and keeps Riley on the same live slot list to pick another, rather than erroring out.

**Success Outcome:** Riley moves their appointment to a time that works, in a couple of taps, with no phone call and no re-paying a deposit.

**Connected Features:** Automated Booking Messaging, Client Booking Identity, Client-Initiated Cancel/Reschedule, Real-Time Slot Availability Engine, Cancellation & No-Show Policy Engine, Two-Way Calendar Sync.

---

### Talia Handles Cancellations and Freed-Up Time

**Owning Persona:** Talia

**Coverage:** Regular

**Journey Goal:** See a client's cancellation reflected correctly (deposit refunded or kept, per policy) and let a waitlisted client claim the newly freed slot.

**Entry Point:** A client cancels their own booking from a reminder text, well inside Talia's cancellation window, and Talia gets a notification.

**Steps:**

1. Cancellation lands — Talia's dashboard shows the booking as cancelled, with the deposit automatically kept because the cancellation fell inside her stated window.
2. Slot reopens — The freed time immediately becomes bookable again on Talia's public page.
3. Waitlist notified — From v1, a different client who had joined the waitlist for that day is notified that a matching slot just opened; at MVP, before the waitlist ships, the slot is simply open to anyone on the public page. [MODIFIED: the waitlist step is marked as v1 to match the Waitlist for Cancelled Slots feature's phase]
4. New booking lands — That client books the freed slot within the 30-minute priority window; Talia's schedule fills back in without her doing anything.

**Failure/Recovery Variant:** No one on the waitlist claims the slot before the priority window expires. The slot simply returns to ordinary public availability, and Talia's schedule shows it as open — nothing is lost or stuck in a pending state.

**Success Outcome:** A cancellation resolves itself correctly (deposit outcome, calendar update, and re-booking opportunity) with no manual intervention from Talia.

**Connected Features:** Client-Initiated Cancel/Reschedule, Cancellation & No-Show Policy Engine, Waitlist for Cancelled Slots, Automated Booking Messaging, Pro Daily Schedule Dashboard.

---

### Resolving a No-Show Dispute

**Owning Persona:** Talia

**Coverage:** Edge/Recovery

**Journey Goal:** Respond confidently to a client who disputes a kept deposit, using a trustworthy record instead of memory or guesswork.

**Entry Point:** A client messages Talia directly (outside the app) disputing why their deposit was kept for a missed appointment.

**Steps:**

1. Talia opens the booking — She finds the booking in question by browsing past bookings on her dashboard and opens its full activity timeline.
2. Timeline review — She sees exactly when the client booked, which cancellation policy version was shown to them and when they ticked to agree to it, the appointment time, and the timestamp she marked it as a no-show.
3. Talia responds — Using the timeline as reference, Talia explains the outcome to the client with confidence, pointing to the exact policy they agreed to; if she decides it was a genuine emergency, she can refund the deposit in full as goodwill instead.
4. Escalation (if needed) — If Talia is unsure how to interpret something, she sends a help request; a support operator opens the same read-only timeline to help, without ever seeing Talia's private notes about the client, and the support view is recorded in Talia's account activity.

**Failure/Recovery Variant:** The client insists the timeline must be wrong and raises a dispute with their card issuer. Talia sees the booking flagged on her dashboard, downloads the plain timeline summary — policy shown and acknowledged, booking time, reminders sent, no-show mark — and submits it through the payment processor as evidence. Because every entry is append-only and immutable, the record itself is the recovery mechanism for the dispute. [AUDIT-ADDED: 1 -- counterpart symmetry: a client can contest a kept deposit through their card issuer, which the draft journey did not cover]

**Success Outcome:** Talia resolves the dispute in minutes with a clear, trustworthy record, rather than losing an evening to an argument she cannot substantiate.

**Connected Features:** No-Show Marking & Deposit Forfeiture, Booking & Payment Activity Record, Pro Daily Schedule Dashboard, Pro Booking Management, Platform Support Read-Only Access, Pro Profile & Booking Page Settings, Cancellation & No-Show Policy Engine.

---

### Recovering from a Declined Deposit Payment

**Owning Persona:** Riley

**Coverage:** Edge/Recovery

**Journey Goal:** Successfully complete a booking after an initial payment attempt fails, without losing the chosen time or having to start over.

**Entry Point:** Riley reaches the payment step of booking and their card is declined.

**Steps:**

1. Decline shown — Riley sees a clear, specific reason for the decline rather than a generic error.
2. Slot still held — Riley's chosen Thursday 2:30pm slot remains held for a short window rather than being released back to the public list immediately.
3. Retry with a different card — Riley re-enters payment details with a different card, without re-selecting the service, time, name, or phone, and without re-agreeing to the policy they already acknowledged.
4. Success — The deposit succeeds on the second attempt, and Riley gets the same confirmation experience as a first-attempt success.

**Failure/Recovery Variant:** Riley takes too long deciding what card to use and the slot hold expires before they retry. Riley sees a plain "that hold has expired, please pick a time again" message and is returned to the live slot list — never charged, and never left uncertain about whether they are booked.

**Success Outcome:** Riley completes their booking despite an initial payment hiccup, with no confusion about whether they were charged or whether their slot was lost.

**Connected Features:** Deposit Payment at Booking, Public Booking Page & Booking Flow, Real-Time Slot Availability Engine.

---

### Talia Cancels a Sick Day

[AUDIT-ADDED: 1 -- counterpart-symmetry walk: no journey covered the Pro being the party who cancels, which BRIEF.md's Target Users & Roles lists as a Pro capability ("reschedule, cancel and refund within policy") and which must never cost a client their deposit]

**Owning Persona:** Talia

**Coverage:** Edge/Recovery

**Journey Goal:** Clear a fully booked day she cannot work, with every client refunded and told, without a single manual refund or DM.

**Entry Point:** Talia wakes up ill with four clients booked for the day and opens Chairtime on her phone.

**Steps:**

1. Block the day — Talia adds a time block for the whole day; Chairtime warns her that four confirmed bookings fall inside it and asks what she wants to do with them.
2. Cancel them together — Talia chooses to cancel all four at once and sees plainly that, because she is the one cancelling, each client's deposit will be refunded in full.
3. Clients told automatically — Each client receives a message (text or email, per their consent) saying Talia had to cancel, their deposit is on its way back, and a link to pick a new time.
4. Check the money — Talia opens her money list and sees the four refunds listed against the deposits they reverse, with nothing taken by Chairtime.
5. Calendar clears — The four appointments disappear from her personal calendar automatically, and the day shows as blocked on her dashboard.

**Failure/Recovery Variant:** One refund cannot complete yet because the deposits have already been paid out to Talia's bank and her processor balance is low. Chairtime flags it on her dashboard as "refund in progress," retries automatically, and tells that client their refund is on its way rather than leaving them wondering; Talia sees it clear in her money list once her next deposit comes in.

**Success Outcome:** Within a couple of minutes, Talia's sick day is cleared, all four clients are refunded and informed, and she can go back to bed without sending a single message.

**Connected Features:** Manual Time Blocking, Pro Booking Management, Cancellation & No-Show Policy Engine, Automated Booking Messaging, Payout Account Connection & Payout Visibility, Two-Way Calendar Sync, Pro Daily Schedule Dashboard.
