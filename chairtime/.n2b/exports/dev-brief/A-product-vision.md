# Part A — Product Vision

This part sets out what Chairtime is and who it is for: the founder's brief, followed by the persona set and access matrix. Read it first; every later part assumes it.

## Executive Summary

Chairtime is a mobile-first booking page for one solo beauty or wellness professional: a client opens the pro's link from an Instagram bio, picks a genuinely free time, pays a card deposit, and receives a confirmation and automatic reminders, while a no-show forfeits the deposit under the pro's own cancellation policy. It serves two product roles, the Pro (owner) and the Client, plus a read-only platform support view. This package contains 30 features specified in 219 specifications carrying 2996 acceptance criteria, a recommended architecture with documented alternatives, and a complete database schema.

## The Brief

The founder's brief, in full, including its open questions (re-surfaced in Part C).


# Chairtime

## Vision
Chairtime is a mobile-first booking page for one independent beauty or wellness professional — a barber, nail tech, lash and brow artist, massage therapist or tattoo artist who rents a chair or works from a home studio. A client opens the pro's link from their Instagram bio, picks a service and a genuinely free time from the pro's real availability, pays a card deposit, and gets a confirmation plus automatic reminders. A no-show means the deposit is kept automatically, under the pro's own cancellation policy that the client agreed to when booking. The pro never negotiates a time in DMs and never chases a deposit again. In the founder's words: "A booking link for one solo beauty pro that takes a card deposit and sends reminders. Client books in a minute; pro never chases a no-show again."

## Problem Statement
Solo beauty and wellness pros lose real money to no-shows and last-minute cancellations, and spend their evenings going back and forth in Instagram DMs to agree on times. Today they negotiate a slot in DMs, hold it in a paper diary or Google Calendar, ask for "$20 on Venmo to hold your spot", and send reminder texts by hand the night before. That falls apart constantly: double bookings, deposits that never arrive, no record when a client disputes a no-show charge, and admin at 11pm. Existing tools are either built for multi-staff salons (heavy setup, a monthly fee for features a solo pro never uses) or free calendar links that can't take a deposit.

## Target Users & Roles
Two product roles, confirmed with the founder. The product is strictly single-operator for v1: a pro with staff, or a salon with several chairs, is explicitly out of scope ("probably forever for this product").

- **The Pro (owner).** A solo barber, nail tech, lash/brow artist, massage therapist or tattoo artist, working from a rented chair or home studio and living on Instagram. The concrete first user is a lash tech or barber the founder personally knows. They currently book via Instagram DMs and ask for Venmo deposits by hand, with about 30 clients a week. The Pro sets up services, prices, the deposit (fixed amount or percentage), working hours, buffer time between clients, and their cancellation window. They see every booking, every client and all the money. They can mark no-shows, reschedule, cancel and refund within policy, and block off time. The Pro is the only person who ever sees the client list, and can delete a client's record on request. They set things up once, then open Chairtime on their phone between clients to see who's next and who has paid.
- **The Client.** Someone who saw the pro's work on Instagram and taps the bio link. The Client books, pays the deposit, reschedules or cancels within the policy window, and sees only their own upcoming and past bookings with that pro. Clients must not face a signup wall or need a password-style account to book; the lightest workable identity is an open question. A client's data is never visible to any other pro or client.
- **Platform operator support access (not a product role).** The founder needs only a read-only support view of a pro's account to help them. There is no client-facing access and nothing more. It is minimal admin access, noted here so downstream work accounts for it.

No other roles.

## The Experience
You're a client who just saw a fresh set of lashes on Instagram, so you tap the link in the pro's bio. It opens right inside Instagram, and on one screen you see the pro's name, their services with prices and how long each takes, and the deposit rule in plain words. You pick "Full set — $65 — 90 min", choose Thursday 2:30pm from times that are genuinely free, enter your name and phone, agree to get texts, pay a $20 deposit by card, and you're done in under a minute. A confirmation text lands immediately. Two days before, a reminder arrives with a one-tap "I'll be there / I need to reschedule." As the pro, you glance at your phone between clients: today's list, each booking with a paid badge, a client note, and how much is still due in person. Nobody negotiated anything. The key moment is the first Friday night you realise you haven't answered a single "are you free Saturday?" DM all week.

## Business Context
Revenue comes from a flat monthly subscription paid by each pro. It is card-based, cancel anytime, with one price tier in v1, priced so that a single saved no-show pays for the month. There is absolutely no per-booking cut: these pros resent that, and it's how they choose tools. Money flow: the client pays the deposit by card at booking, and it goes to the pro (the payment processor handles payout to the pro). The balance is due at the appointment. A no-show or a cancellation inside the pro's window forfeits the deposit to the pro, and a cancellation outside the window refunds the deposit automatically. The platform takes no cut of any of it. Why now: many of these pros went solo in the last few years and live on Instagram, paying a card deposit to a small business is now normal, and existing tools are either salon-sized or free-and-dumb. Go-to-market is the founder's own network of pros plus peer referral.

## Scale & Non-Functional Expectations
- **Volume:** a few hundred pros in year one. Each pro has roughly 100–500 clients and 20–40 bookings a week.
- **Devices & platforms:** mobile-first web for both roles. The pro checks it on their phone between clients, and the client books from inside the Instagram in-app browser, so it must work well there. Desktop is a bonus for the pro's setup screens.
- **Geography:** US first; the UK, Canada and Australia are the obvious next markets, so timezone and currency must not be hard-coded.
- **Correctness & reliability:** reliability matters more than features. Booking and payment must be correct, always. It must never silently double-book or lose a deposit; if it does, the pro leaves and tells their friends. No specific uptime number was given.
- **Privacy:** a client's data is visible only to their pro (and to the client themselves for their own bookings), never to any other pro or client. A pro can delete a client's record on request.

## Ecosystem & Integrations
- **Pro's personal calendar (Google Calendar and Apple Calendar — both matter):** two-way. Busy times there block Chairtime availability, and bookings made in Chairtime appear there.
- **SMS text messaging:** confirmations and reminders by text in v1, sent only with the client's explicit opt-in at booking. Email confirmations are acceptable as a fallback. WhatsApp is a nice-to-have later, not v1.
- **Established card payment processor:** takes client deposits and pays out to the pro, and owns all card data. Also used for the pro's subscription billing.
- **Instagram:** only the place the pro's booking link lives. No Instagram integration for v1.
- Otherwise it stands alone.

## Success Criteria
One year from now, it worked if:
- Pros say "I haven't had an unpaid no-show since I switched".
- Pros stop taking bookings by DM entirely — the link is the only way to book them.
- Most new pros arrive because another pro told them about it.
- Nobody has ever had a double booking or a lost deposit.

## Constraints
- **Team / timeline:** a solo founder building with AI coding tools, wanting the first paying pro within about three months. *Rationale:* one-person team and speed to first revenue.
- **Budget:** infrastructure under roughly $100/month until there's revenue. *Rationale:* pre-revenue solo venture.
- **Payments (regulatory / security):** card data is never stored or handled by the founder's code; the payment processor owns it. *Rationale:* "I never want to see or store a card number."
- **Messaging consent (regulatory):** clients must explicitly agree to receive texts when they book, and reminders must respect that consent. *Rationale:* US texting rules.
- **Personal data (regulatory / privacy):** a pro must be able to delete a client's record on request, and a client's data is never visible to any other pro or client. *Rationale:* client privacy and deletion-on-request obligations.
- **Platform:** mobile-first web only for v1; no native apps, no app stores. *Rationale:* where users are (phone, Instagram in-app browser) and speed to launch.
- **Scope:** strictly single-operator for v1; multi-staff pros and multi-chair salons are out of scope. *Rationale:* that's a different product.
- **Regulated-domain confirmation:** payments, US SMS consent and personal-data deletion were confirmed as the regulated areas. No health data: massage/tattoo intake forms are out of scope for v1.
- **Technology:** no technology commitments. *Rationale:* build it with whatever the blueprint recommends.

## Open Questions
- **Balance payment:** should the balance after the deposit be payable in the app, or stay in person (cash / card reader at the chair)?
- **Waitlist:** should a pro be able to offer a waitlist for slots that open up from cancellations?
- **Tipping:** where does tipping fit, if anywhere?
- **Client identity:** what is the lightest workable identity for clients so they can manage their own bookings without a signup wall? A phone number plus a magic link, or something else?
- **Recurring appointments:** are recurring or standing appointments ("every 3 weeks") v1 or later?
- **Subscription price point:** the model is a flat monthly fee "cheap enough that a single saved no-show pays for the month", but the actual price was not stated.
- **Availability target:** there is no stated uptime number; the requirement is correctness ("never silently double-book, never lose a deposit"). Downstream should decide what availability that implies.

## Source Materials
- `.n2b/inputs/source/pasted-notes.md` — the founder's full written napkin brief (problem, roles, experience, business, scale, integrations, success criteria, hard boundaries, open questions), preserved verbatim.

*Preserved verbatim so nothing the user wrote is lost in this brief's compression.
Stage 2 works from this brief (brief-first); the originals are kept for reference and
for Stage 1 re-runs.*


## Who This Is For

The persona set: primary and secondary personas and the role Access Matrix.


# User Personas

## Persona Set Summary

This product serves two genuine product roles plus one narrow non-product support role. The Pro is the primary persona and the commercial user; the Client is a secondary persona who books but never holds an account in the traditional sense. A third, lightweight role — Platform Operator (Support) — exists solely for read-only troubleshooting access, per BRIEF.md's Target Users & Roles section, which states there are "no other roles." Market research surfaced no further role this product needs: the multi-staff and payroll roles common in competitor tools reflect salon-scale products, which BRIEF.md explicitly rules out. [RESEARCH-INFORMED: role patterns in the profiled competitors are salon/multi-staff-oriented and are a market pattern only, not evidence that a single-operator product needs more roles, from the market-research Feature Landscape (4 of 5 profiles)]

## Primary Persona

### Persona Name

Talia

### Description

Talia is a solo lash and brow artist who rents a chair inside a shared studio. She built her clientele almost entirely through Instagram — her feed is her portfolio and her booking desk. She is not a technical person and has no interest in "software"; she wants a link she can drop in her Instagram bio that just works. Today she negotiates every appointment in Instagram DMs, holds her schedule in her head and a paper notebook, and asks new clients to send "$20 on Venmo to hold the spot" — a system that leaks money and eats her evenings. (BRIEF.md, Target Users & Roles; Problem Statement.)

### Goals

- Stop negotiating times in Instagram DMs entirely — the link does that job (BRIEF.md, The Experience: "Nobody negotiated anything.")
- Never lose money to a no-show again — the deposit is collected automatically, lands with her directly, and is kept automatically under her own policy (BRIEF.md, Business Context) [MODIFIED: "lands with her directly" added to reflect the audit-added Payout Account Connection & Payout Visibility feature]
- See her day at a glance between clients — who's next, who's paid, what's still owed (BRIEF.md, The Experience)
- Set up once (services, prices, deposit rule, hours, buffer time, cancellation window) and then barely think about admin again (BRIEF.md, Target Users & Roles)
- Keep her existing personal calendar (Google or Apple) as the one place her whole life's schedule lives, with Chairtime never conflicting with it (BRIEF.md, Ecosystem & Integrations)

### Pain Points

- DM negotiation is slow, happens at all hours, and often stalls before a time is even agreed (BRIEF.md, Problem Statement: "admin at 11pm")
- Venmo deposits are asked for by hand and frequently never arrive, so there is no reliable hold on a slot (BRIEF.md, Problem Statement)
- No record exists when a client disputes a no-show charge — it becomes her word against theirs (BRIEF.md, Problem Statement)
- Existing booking tools are built for multi-staff salons, with setup and fees for features she will never use, or are free calendar links that cannot take a deposit at all (BRIEF.md, Problem Statement) [RESEARCH-INFORMED: layered, hard-to-predict fees on top of the advertised subscription are the dominant pricing complaint across three competitors (BBB, Capterra and Reddit-derived sources, HIGH confidence), and per-new-client marketplace commissions of 20–30% are widely resented (MEDIUM–HIGH confidence)]
- Double bookings happen because her diary/calendar and her DM-agreed times are two systems that never talk to each other (BRIEF.md, Problem Statement) [RESEARCH-INFORMED: even dedicated booking tools are reported to glitch in the scheduling/calendar workflow at her volume of dozens of bookings a week, from aggregated user reviews of two competitors (MEDIUM confidence)]

### Behavioral Context

Talia does nearly all of her Chairtime use on her phone, in short bursts between clients — checking who's next, confirming a payment landed, marking a no-show, rebooking a regular before they leave the chair. Setup (services, hours, cancellation policy, payout account) happens in a longer, one-time session, for which a desktop screen is a welcome bonus but not required (BRIEF.md, Scale & Non-Functional Expectations: "mobile-first web for both roles... Desktop is a bonus for the pro's setup screens"). She checks Chairtime reactively throughout a working day rather than on a fixed schedule. [MODIFIED: rebooking at the chair and payout setup added to reflect the audit-added Pro Booking Management and Payout Account Connection & Payout Visibility features]

### What This User Does NOT Need

- Staff scheduling, multi-chair management, or any concept of "team" — the product is strictly single-operator (BRIEF.md, Constraints: "strictly single-operator for v1")
- Health or medical intake forms — massage and tattoo intake is explicitly out of scope for v1 (BRIEF.md, Constraints)
- Any exposure to card numbers or payment credentials — she never wants to see or store one (BRIEF.md, Constraints: "I never want to see or store a card number")
- Instagram integration beyond having a link to share — no DM automation, no feed posting (BRIEF.md, Ecosystem & Integrations: "No Instagram integration for v1")
- A native app or app-store install — she wants something that works the moment a client taps a link (BRIEF.md, Constraints: "no native apps, no app stores")
- A marketplace that sends her strangers in exchange for a cut of each new client — her clients find her through her own Instagram (BRIEF.md, Business Context: "absolutely no per-booking cut") [RESEARCH-INFORMED: marketplace discovery in two competitors is tied to a one-off 20–30% new-client commission that professionals widely complain about (MEDIUM–HIGH confidence)]

## Secondary Personas

### The Client

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section names the Client as a distinct role with its own goals, constraints ("must not face a signup wall"), and privacy boundary ("never visible to any other pro or client"), which differ entirely from the Pro's entitlements.

**Name:** Riley

**Description:** Riley saw a fresh set of lashes on Instagram and tapped the artist's bio link. Riley is not signing up for a "platform" — Riley wants to book one appointment with one person, quickly, from inside the Instagram in-app browser, without creating a password-protected account (BRIEF.md, The Experience; Target Users & Roles).

**Goals:**
- Book an appointment in under a minute without leaving the Instagram app experience (BRIEF.md, The Experience)
- Know exactly what the deposit rule is — and exactly when the cancellation window closes for this booking — before paying anything (BRIEF.md, The Experience: "the deposit rule in plain words") [RESEARCH-INFORMED: the moment of policy disclosure at booking is where client trust is lost in this market, from BBB complaint records and forum-derived summaries (MEDIUM confidence)]
- Get a clear confirmation (including where to go) and a helpful reminder, and be able to reschedule or cancel without hunting for a phone number (BRIEF.md, The Experience; Target Users & Roles)
- See only their own upcoming and past bookings with this one pro — nothing more (BRIEF.md, Target Users & Roles)

**Pain Points:**
- Booking by DM means waiting for a reply and negotiating back and forth before a time is even confirmed (BRIEF.md, Problem Statement)
- No current lightweight way to prove or manage a booking without a full account (BRIEF.md, Open Questions: "Client identity")
- Being surprised afterward by a kept deposit or cancellation charge whose terms were not clear when booking [RESEARCH-INFORMED: frequently mentioned client-side complaint about non-refundable deposits and disputed cancellation charges on one competitor, from Trustpilot/BBB complaint records (MEDIUM confidence)]

**Behavioral Context:** Riley books from a phone, inside Instagram's in-app browser, in a single short session triggered by seeing the pro's work on their feed or story. Riley returns briefly around reminder time (to confirm or reschedule) and, occasionally, to book again with the same pro later — sometimes by scanning a deposit link on the pro's screen at the end of an appointment.

**What This User Does NOT Need:** An account with a password; visibility into the pro's other clients or bookings; access to any other pro's booking page or client data (BRIEF.md, Target Users & Roles, Constraints: personal-data privacy); a card kept on file between visits — each deposit is paid fresh (BRIEF.md, Constraints: card data owned by the payment processor).

### Platform Operator (Support)

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section explicitly names this as "Platform operator support access (not a product role)... a read-only support view of a pro's account," included "so downstream work accounts for it."

**Name:** N/A — this is the founder/operator function, not a persona to design experiences around.

**Description:** The founder, in an operational capacity, occasionally needs to look into a specific pro's account to help them resolve a problem (a missed payment, a confusing booking, a support request, a client dispute).

**Goals:** Diagnose and help resolve a pro's account issue without needing the pro to screen-share or describe every setting verbally; answer support requests quickly. [RESEARCH-INFORMED: slow, email-only support is a frequently mentioned complaint about three competitors (MEDIUM confidence), so quick diagnosis is a trust asset]

**Pain Points:** N/A — this is an internal operational need, not a user pain point the product is designed to solve.

**Behavioral Context:** Occasional, on-demand use triggered by a Pro's help request; never a daily or scheduled workflow. Every support view is recorded in the Pro's account activity, which the Pro can see. [MODIFIED: support views are now logged and visible to the Pro, per the audit-logging addition to Platform Support Read-Only Access]

**What This User Does NOT Need:** Any client-facing capability, any ability to edit a pro's data, bookings, or payments, any ability to sign in as a Pro or see sign-in codes, bank or identity details, or the Pro's private client notes, and no access beyond what is needed to view and diagnose (BRIEF.md, Target Users & Roles: "There is no client-facing access and nothing more. It is minimal admin access").

## Access Matrix

[MODIFIED: extended from 6 to 12 capability groups so the matrix covers the audit-added features (profile and account settings, payouts, Pro booking management) and the draft features whose per-feature Access fields had no matching column (activity record and insights, waitlist, recurring appointments, support access); cell values for existing columns are unchanged]

| Role / Persona | Service & Availability Setup | Booking & Payment | Client Records | Cancellation & No-Show Handling | Messaging & Consent | Subscription & Billing | Profile & Account Settings | Payouts | Activity Record & Insights | Waitlist | Recurring Appointments | Support Access Log |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| The Pro (Talia) | Full | Full | Full | Full | Full | Full | Full | Full | View | View | Full | View |
| The Client (Riley) | None | Own-only | None | Own-only | Own-only (their own consent) | None | View (public profile only) | None | None | Own-only | Own-only | None |
| Platform Operator (Support) | View | View | View (read-only, for troubleshooting only; never the Pro's private notes) | View | View | View | View (status only; never sign-in codes) | View (status and money list only; never bank or identity details) | View | View | View | View |

**How to read the groups (feature mapping):**
- **Service & Availability Setup:** Service & Pricing Management, Availability & Working Hours Setup, Real-Time Slot Availability Engine (setup side), Two-Way Calendar Sync, Manual Time Blocking.
- **Booking & Payment:** Public Booking Page & Booking Flow, Client Booking Identity, Deposit Payment at Booking, Pro Daily Schedule Dashboard, Pro Booking Management, In-App Balance Payment, Tipping at Checkout. The Client's Own-only access runs entirely through Client Booking Identity; Support views bookings but never uses or bypasses a client's access links.
- **Client Records:** Client Record Management, Client List Search & Filter.
- **Cancellation & No-Show Handling:** Cancellation & No-Show Policy Engine, Client-Initiated Cancel/Reschedule, No-Show Marking & Deposit Forfeiture.
- **Messaging & Consent:** Automated Booking Messaging, Messaging Consent Management, WhatsApp Reminders. The Pro's Full access covers their own notifications and message history; the Pro can see but never override a client's texting consent.
- **Subscription & Billing:** Pro Subscription Billing & Account Management.
- **Profile & Account Settings:** Pro Onboarding & Setup Wizard, Pro Profile & Booking Page Settings, Pro Sign-In & Account Lifecycle. The Client sees only the public profile fields on the booking page and the studio address in their own confirmation.
- **Payouts:** Payout Account Connection & Payout Visibility.
- **Activity Record & Insights:** Booking & Payment Activity Record, Booking & Revenue Insights. View for everyone: the activity record is append-only, so nobody — including the Pro — can edit it.
- **Waitlist:** Waitlist for Cancelled Slots. The Pro sees demand; only clients join or leave.
- **Recurring Appointments:** Recurring/Standing Appointments.
- **Support Access Log:** Platform Support Read-Only Access. Support has View access to one Pro account at a time; the Pro sees the log of when support looked.

**Unauthorized access:** Anyone who is not signed in as the Pro and tries to reach a Pro screen is sent to the Pro sign-in screen; a client without a valid link for a booking sees only a "request a new link" prompt; a visitor to a closed or mistyped booking link sees a plain "this booking page isn't available" message. No role can ever see another pro's data or another client's bookings.

