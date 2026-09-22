---
document_type: user-journeys
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-21
coherence_check: passed
---

# User Journeys

## Journey

This document contains 8 journeys, covering the required minimum for 29 defined features. Taylor (the Client) owns 3 journeys; Mara (the Pro) owns 5. Coverage spans first-use (2), regular use (3), and edge/recovery (3), with all three values represented at least once. Every Core and Important feature from product-features.md appears in at least one journey below.

---

### Client First Booking

**Owning Persona:** Taylor

**Coverage:** First-use

**Journey Goal:** Go from tapping a Pro's Instagram bio link to a confirmed, deposit-paid appointment, in under a minute, without ever DMing anyone.

**Entry Point:** Taylor taps the booking link in Mara's Instagram bio, opened inside Instagram's own in-app browser.

**Steps:**

1. Land on the booking page — Taylor sees Mara's name, her services with prices and durations, and the deposit rule stated plainly. No login screen stands between Taylor and browsing.
2. Pick a service and time — Taylor selects "Full set — $65 — 90 min" and is shown only genuinely free times; Taylor picks Thursday 2:30pm, which is held for the moment.
3. Provide identity and consent — Taylor enters name and phone, verifies with a one-time code, and explicitly agrees to texts and the cancellation policy.
4. Pay the deposit — Taylor pays a $20 deposit by card; the product itself never touches the card number.
5. Confirmation — Taylor sees the booking confirmed and a confirmation text lands immediately. The whole flow took under a minute.

**Failure/Recovery Variant:** While Taylor is entering booking details, another client completes payment for the same 2:30pm slot first. Taylor's payment step shows an immediate, plain "that time was just booked — please pick another" message rather than a failed charge, and Taylor is returned to slot selection with the rest of their entered details preserved.

**Success Outcome:** Taylor has a confirmed, deposit-paid appointment and a confirmation text, with no negotiation and no DM ever sent.

**Connected Features:** Public Booking Page, Live Availability & Slot Booking, Client Identity & Booking Details Capture, Deposit Payment at Booking, Booking Confirmation & Reminders, SMS Messaging & Consent Capability, Payment Processing Capability

---

### Pro Onboarding & Setup

**Owning Persona:** Mara

**Coverage:** First-use

**Journey Goal:** Go from signing up to having a working, shareable booking page ready to put in her Instagram bio.

**Entry Point:** Mara signs up for the product for the first time, motivated to stop running her diary by DM.

**Steps:**

1. Create account — Mara signs up and creates her business profile with her name, timezone, and currency.
2. Add services and policy — Mara adds her services with prices and durations, and sets her deposit amount and cancellation window.
3. Set hours — Mara sets her weekly working hours and the buffer time she needs between clients.
4. Connect calendar (optional) — Mara connects her personal calendar so her existing commitments are respected from day one, or she skips this and does it later.
5. Get her link — Mara receives her bio-link URL, ready to paste into her Instagram bio.

**Failure/Recovery Variant:** Mara closes the product midway through adding services, distracted by a client. When she reopens it later that day, setup resumes exactly where she left off — her business name, timezone, and any services already added are preserved, and she is not asked to start over.

**Success Outcome:** Within one session (resumed if interrupted), Mara has at least one bookable service, a policy, working hours, and a live link she can put in her bio.

**Connected Features:** Pro Onboarding & Setup, Pro Account & Authentication, Business Settings & Policy Configuration, Two-Way Calendar Sync, Pro Subscription & Billing

---

### Pro's Daily Chair-Side Workflow

**Owning Persona:** Mara

**Coverage:** Regular

**Journey Goal:** Know exactly what to expect from each client today, glancing at her phone between appointments, without any manual reconciliation.

**Entry Point:** Mara has a moment between clients and checks her phone, as she does several times most working days.

**Steps:**

1. Open today's list — Mara sees every remaining booking for today, in time order.
2. Check paid status — Each booking shows a clear paid badge, so Mara never wonders who has actually paid.
3. Read the client note — Mara sees any note she's kept on that client (a preference, an allergy, a history detail) alongside the booking.
4. Confirm balance due — Mara sees exactly how much is still owed at the chair for the next client.
5. Block unplanned time, if needed — If a personal matter comes up, Mara blocks the rest of the afternoon in a couple of taps so it stops appearing as bookable.

**Failure/Recovery Variant:** A client reschedules from their reminder text while Mara is mid-appointment with someone else. When Mara next checks her phone, today's list already reflects the change — she is never working from a stale list or double-booked by a change she didn't see happen.

**Success Outcome:** Mara runs her whole day off one trustworthy list, with no DM checking and no separate reconciliation against a paper diary.

**Connected Features:** Pro Daily Dashboard, No-Show & Cancellation Deposit Handling, Pro Manual Schedule Blocking, Client Record Management

---

### Client Reminder-Driven Reschedule

**Owning Persona:** Taylor

**Coverage:** Regular

**Journey Goal:** Respond to a reminder and change plans without a phone call or a DM.

**Entry Point:** Two days before the appointment, a reminder text arrives with a one-tap "I'll be there / I need to reschedule."

**Steps:**

1. Receive the reminder — Taylor gets the reminder text and reads the one-tap choice.
2. Realize plans changed — Taylor taps "I need to reschedule" instead of confirming.
3. Pick a new time — Taylor is shown live availability and picks a new genuinely free slot for the same service.
4. See the outcome plainly — Because the change is inside the cancellation window, Taylor sees the reschedule confirmed with no new charge.
5. Get a new confirmation — A confirmation text arrives for the new time.

**Failure/Recovery Variant:** Taylor tries to reschedule the same day the reminder arrives, which is outside Mara's cancellation window. Before confirming, Taylor is shown plainly that this counts as a late change and the deposit will be forfeited if cancelled outright — so Taylor can still choose to reschedule to a nearby time without penalty, or accept the forfeiture if cancelling entirely. Nothing is a surprise after the fact.

**Success Outcome:** Taylor's appointment moves to a time that works, entirely self-service, with the policy outcome always visible before it's final.

**Connected Features:** Booking Confirmation & Reminders, Client Self-Service Reschedule & Cancellation, Client Self-Service Booking History

---

### Pro Adjusts Policy and Reviews Business Health

**Owning Persona:** Mara

**Coverage:** Regular

**Journey Goal:** Tighten her cancellation policy after a rough patch and confirm, in her own numbers, that the change is working.

**Entry Point:** Mara notices a few too many late cancellations in a row and decides to act, outside of any specific appointment.

**Steps:**

1. Open business settings — Mara reviews her current deposit amount and cancellation window.
2. Tighten the policy — Mara increases the cancellation window from 24 to 48 hours; the change applies to new bookings going forward.
3. Confirm her subscription is in good standing — Mara glances at her billing status while she's in settings.
4. Check her weekly snapshot — A few weeks later, Mara opens her simple business insights and sees her no-show rate has dropped.
5. Feel the difference — Mara notices she hasn't answered a single "are you free Saturday?" DM in weeks.

**Failure/Recovery Variant:** Mara accidentally sets the cancellation window shorter than she meant to (an easy mis-tap). Because the change only affects new bookings — never retroactively changing terms already agreed to by existing clients — the mistake is low-stakes, and she simply corrects the number the next time she opens settings.

**Success Outcome:** Mara's policy reflects what she's actually experiencing, and she has plain evidence her situation has improved, without doing any manual math.

**Connected Features:** Business Settings & Policy Configuration, Pro Subscription & Billing, Simple Business Insights

---

### No-Show and Deposit Forfeiture

**Owning Persona:** Mara

**Coverage:** Edge/Recovery

**Journey Goal:** Protect her income when a client simply doesn't show up, and have something concrete if it's later disputed.

**Entry Point:** The appointment time passes and the client hasn't arrived or messaged.

**Steps:**

1. Wait past the appointment time — Mara gives it a few minutes, as she normally would.
2. Mark the no-show — Mara taps "no-show" on that booking from her daily dashboard.
3. Deposit stays put — The deposit is retained automatically; Mara does nothing further to keep it.
4. Client disputes later — A week later, the client messages claiming they were never properly booked or charged.
5. Pull up the record — Mara opens that booking's history and sees the exact policy the client agreed to, the payment timestamp, and the no-show timestamp.

**Failure/Recovery Variant:** Mara realizes minutes later she tapped "no-show" on the wrong booking by mistake. Because the mark can be undone shortly after (before it's included in any settlement), she corrects it immediately without needing support intervention — though if she genuinely got stuck, the founder's read-only support console could look at the same booking to help confirm what happened.

**Success Outcome:** Mara keeps the deposit she's owed and has a concrete, timestamped answer the one time a client pushes back — instead of no record at all.

**Connected Features:** No-Show & Cancellation Deposit Handling, Booking Record & Dispute Trail, Pro Daily Dashboard, Operator Support Console

---

### Booking Conflict Recovery via Calendar Sync

**Owning Persona:** Mara

**Coverage:** Edge/Recovery

**Journey Goal:** Trust that a personal commitment on her own calendar never turns into a double-booking, even when sync briefly lags.

**Entry Point:** Mara adds a personal appointment (a dentist visit) directly to her own connected calendar, expecting it to protect that time here too.

**Steps:**

1. Add the personal event — Mara books her dentist appointment on her own calendar, as she always has.
2. Sync picks it up — Within a few minutes, that time stops appearing as bookable on her availability.
3. A client almost books the same time — Before sync completed, a client had the old availability open; when they try to confirm, the slot is no longer offered.
4. Mara adds a manual block, just in case — For anything time-sensitive, Mara also uses manual blocking as a direct, immediate backstop rather than relying on sync alone.
5. Confidence restored — No double-booking occurs, and Mara trusts the system even when sync has a short delay.

**Failure/Recovery Variant:** The calendar connection becomes briefly unreachable. Availability falls back to internally-known bookings and manual blocks only, with a discreet notice to Mara that externally-blocked time may be briefly stale — rather than confidently showing a slot that's actually taken. Mara uses manual blocking to cover anything urgent until sync recovers.

**Success Outcome:** Double-booking never silently happens, even through a sync delay or outage — the non-negotiable promise holds.

**Connected Features:** Two-Way Calendar Sync, Live Availability & Slot Booking, Pro Manual Schedule Blocking

---

### Client Data Deletion Request

**Owning Persona:** Mara

**Coverage:** Edge/Recovery

**Journey Goal:** Honor a client's request to have their personal data removed, cleanly and completely.

**Entry Point:** A client messages Mara (outside the product, as this is a personal request) asking her to delete their information.

**Steps:**

1. Find the client — Mara searches her client list by name or phone and finds the record.
2. Confirm the request — Mara opens the client's record to confirm it's the right person before deleting.
3. Delete the record — Mara deletes the client's personal details; past booking history is anonymized rather than silently vanishing from her own financial records.
4. Confirm to the client — An optional acknowledgment lets the client know their data was removed.
5. List reflects the change — The client no longer appears in Mara's client list.

**Failure/Recovery Variant:** Mara starts the deletion but hesitates, unsure if it's fully reversible. The deletion flow requires a deliberate, explicit confirmation step (not a single accidental tap) precisely because it is largely irreversible, giving Mara a clear moment to back out before anything is actually removed.

**Success Outcome:** The client's personal data is genuinely gone from Mara's records, honoring the brief's privacy promise, without disrupting Mara's own financial history.

**Connected Features:** Client Record Management, Account Closure & Client Data Deletion

