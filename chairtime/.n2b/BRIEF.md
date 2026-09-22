---
project_name: Chairtime
domain: appointment booking and no-show protection for solo beauty & wellness professionals
created: 2026-09-22
status: active
n2b_version: 0.4.0
---

# Chairtime

## Vision
Chairtime is a mobile-first, deposit-first booking page for one independent beauty or wellness professional — a barber, nail tech, lash or brow artist, massage therapist, or tattoo artist who rents a chair or works from a home studio and runs their business out of Instagram DMs. A client opens the pro's link, picks a service and a genuinely free time, pays a card deposit under the pro's own cancellation policy, and gets automatic confirmation and reminders. The pro gets a clean view of their day, sees who has paid, and never chases anyone; a no-show keeps the deposit. It exists so a solo pro stops losing money to no-shows and stops running their diary by DM at 11pm.

## Problem Statement
Solo beauty and wellness pros lose real money to no-shows and last-minute cancellations, and spend their evenings negotiating slots in Instagram DMs. Today they agree a time by DM, hold it in a paper diary or Google Calendar, ask for a "send me $20 on Venmo to hold your spot" deposit, and text reminders by hand the night before. This falls apart constantly: double bookings, deposits that never arrive, no record when a client disputes a no-show charge, and admin done late at night. Existing tools are either built for multi-staff salons (heavy setup, monthly fees for features a solo pro never uses) or free calendar links that cannot take a deposit.

## Target Users & Roles
Two product roles, confirmed by the founder. Strictly single-operator for v1: a pro with staff, or a salon with several chairs, is explicitly out of scope and a different product.

- **The Pro (owner).** An independent operator such as a lash artist working from a home studio or a barber renting a chair, who finds clients on Instagram. Sets up services, prices, deposit amount or percentage, working hours, buffer time between clients, and their cancellation window. Sees every booking, every client, and all the money. Can mark no-shows, reschedule, cancel and refund within policy, and block off time. Reaches for the product between clients on their phone to check today's list, and when setting policy. The Pro is the only person who ever sees the client list.
- **The Client.** Someone who found the pro on Instagram and taps the bio link. Books, pays the deposit, reschedules or cancels within the policy window, and sees only their own upcoming and past bookings with that pro. Must not need a password-style account just to book; the lightest workable identity is an open question. Reaches for the product when they want an appointment and when a reminder arrives.
- **Platform operator support (narrow actor, not a product role).** The founder, as operator, needs a read-only look at a pro's setup and bookings to troubleshoot — never acting on the pro's behalf and never seeing more of client data than the pro's own screens show. Confirmed as read-only in conversation.

No other roles exist. Clients' data is never visible to any other pro or client.

## The Experience
You tap the link in the pro's Instagram bio and, without leaving Instagram's in-app browser, you see the pro's name, their services with prices and durations, and the deposit rule in plain words. You pick "Full set — $65 — 90 min", pick Thursday 2:30pm from times that are genuinely free, enter your name and phone, agree to texts and the cancellation policy, pay a $20 deposit by card, and you're done in under a minute. A confirmation text lands immediately. Two days before, a reminder arrives with a one-tap "I'll be there / I need to reschedule". On the other side, the pro glances at their phone between clients and sees today's list, each booking with a paid badge, a client note, and how much is still due at the chair; a client who doesn't turn up gets one tap to mark the no-show and the deposit stays put, and a cancellation outside the window forfeits on its own. Nobody negotiated anything. The key moment is the first Friday night the pro realises they haven't answered a single "are you free Saturday?" DM all week.

## Business Context
Commercial: a flat monthly subscription per pro, priced cheaply enough that a single saved no-show pays for the month. Absolutely no per-booking cut — these pros resent it and it is how they choose tools. Money flow, confirmed end-to-end: the client's deposit is collected by the card processor straight into the pro's own payout account (processor fees on the pro's side); the pro refunds within their policy; a no-show or out-of-window cancellation forfeits the deposit to the pro; the pro pays the subscription by card inside the product; the platform never holds client money. The balance after the deposit is settled in person today — whether it becomes payable in-app is an open question. Why now: a huge number of these pros went solo in the last few years and live on Instagram, paying a card deposit to a small business is now normal, and the existing tools are either salon-sized or free-and-dumb. First pros come from the founder's own network (several known personally) and from those pros referring peers.

## Scale & Non-Functional Expectations
- **Usage:** a few hundred pros in the first year; each pro has roughly 100–500 clients and 20–40 bookings a week.
- **Platforms:** mobile-first web for both sides. The pro checks it on their phone between clients; the client books from inside the Instagram in-app browser, so it must work well there. No native apps or app stores in v1.
- **Geography:** start in the US; timezone and currency must not be hard-coded because the UK, Canada and Australia are the obvious next markets.
- **Reliability over features:** the moment it silently double-books or loses a deposit, the pro is gone and tells their friends. Double-booking integrity and deposit integrity are the non-negotiable qualities.
- **Privacy:** client data is visible only to the one pro it belongs to; a pro must be able to delete a client's record on request.
- **Performance and availability targets:** not stated beyond "reliability matters more than features" — flagged as open question.

## Ecosystem & Integrations
- **The pro's personal Google Calendar or Apple Calendar** — two-way: busy times there block availability here, and bookings made here appear there.
- **Text-message reminders (SMS)** — confirmation and reminders to clients, with explicit consent captured at booking. WhatsApp is a later nice-to-have, not v1.
- **An established card payment processor** — collects deposits and subscription payments; owns all card data. The product never sees or stores a card number.
- **Instagram** — only the place the link lives; no Instagram integration in v1.
- Otherwise standalone (confirmed).

## Success Criteria
One year in, it worked if:
- Pros say "I haven't had an unpaid no-show since I switched".
- Pros stop taking bookings by DM entirely — the link is the only way to book them.
- Most new pros arrive because another pro told them about it.
- Nobody has ever had a double booking or a lost deposit.

## Constraints
- **Team (resourcing):** solo founder building with AI coding tools — the blueprint must be buildable by one person.
- **Timeline:** first paying pro within about three months.
- **Budget:** infrastructure under roughly $100/month until there is revenue.
- **Payments (regulatory / security):** card data is never stored or handled by the product's own code — the processor owns it.
- **Messaging (regulatory):** clients must explicitly agree to receive texts when they book, and reminders must respect that consent (US texting rules).
- **Personal data (privacy):** a pro must be able to delete a client's record on request; clients' data is never visible to any other pro or client.
- **Platform:** mobile-first web only for v1; no native apps, no app stores.
- **Technology:** no commitments — build with whatever the blueprint recommends.
- **Scope:** single-operator only; multi-staff or multi-chair is out of scope.

The constraints question was asked once; the founder confirmed the list above is complete.

## Open Questions
- Should the balance after the deposit be payable in the app, or does it stay in person (cash / card reader at the chair)?
- Should a pro be able to offer a waitlist for slots that open up from cancellations?
- Where does tipping fit, if anywhere?
- What is the lightest identity for clients — phone number plus a magic link, or something else? No signup wall.
- Recurring / standing appointments ("every 3 weeks") — v1 or later?
- Performance and availability targets beyond "never silently double-book or lose a deposit" — not stated; research should propose what a solo pro's business tolerates.

## Source Materials
- `.n2b/inputs/source/napkin.md` — the founder's original written brief covering problem, product, roles, experience, business, scale, integrations, success, boundaries and open questions.

*Preserved verbatim so nothing the user wrote is lost in this brief's compression.
Stage 2 works from this brief (brief-first); the originals are kept for reference and
for Stage 1 re-runs.*
