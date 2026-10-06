---
project_name: Chairtime
domain: appointment booking and no-show protection for solo beauty and wellness professionals
created: 2026-09-26
status: active
n2b_version: 0.4.0
---

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
