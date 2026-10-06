---
document_type: user-persona
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# User Personas

## Persona Set Summary

This product serves two genuine product roles plus one narrow non-product support role. The Pro is the primary persona and the commercial user; the Client is a secondary persona who books but never holds an account in the traditional sense. A third, lightweight role — Platform Operator (Support) — exists solely for read-only troubleshooting access, per BRIEF.md's Target Users & Roles section, which states there are "no other roles."

## Primary Persona

### Persona Name

Talia

### Description

Talia is a solo lash and brow artist who rents a chair inside a shared studio. She built her clientele almost entirely through Instagram — her feed is her portfolio and her booking desk. She is not a technical person and has no interest in "software"; she wants a link she can drop in her Instagram bio that just works. Today she negotiates every appointment in Instagram DMs, holds her schedule in her head and a paper notebook, and asks new clients to send "$20 on Venmo to hold the spot" — a system that leaks money and eats her evenings. (BRIEF.md, Target Users & Roles; Problem Statement.)

### Goals

- Stop negotiating times in Instagram DMs entirely — the link does that job (BRIEF.md, The Experience: "Nobody negotiated anything.")
- Never lose money to a no-show again — the deposit is collected automatically and forfeited automatically under her own policy (BRIEF.md, Business Context)
- See her day at a glance between clients — who's next, who's paid, what's still owed (BRIEF.md, The Experience)
- Set up once (services, prices, deposit rule, hours, buffer time, cancellation window) and then barely think about admin again (BRIEF.md, Target Users & Roles)
- Keep her existing personal calendar (Google or Apple) as the one place her whole life's schedule lives, with Chairtime never conflicting with it (BRIEF.md, Ecosystem & Integrations)

### Pain Points

- DM negotiation is slow, happens at all hours, and often stalls before a time is even agreed (BRIEF.md, Problem Statement: "admin at 11pm")
- Venmo deposits are asked for by hand and frequently never arrive, so there is no reliable hold on a slot (BRIEF.md, Problem Statement)
- No record exists when a client disputes a no-show charge — it becomes her word against theirs (BRIEF.md, Problem Statement)
- Existing booking tools are built for multi-staff salons, with setup and fees for features she will never use, or are free calendar links that cannot take a deposit at all (BRIEF.md, Problem Statement)
- Double bookings happen because her diary/calendar and her DM-agreed times are two systems that never talk to each other (BRIEF.md, Problem Statement)

### Behavioral Context

Talia does nearly all of her Chairtime use on her phone, in short bursts between clients — checking who's next, confirming a payment landed, marking a no-show. Setup (services, hours, cancellation policy) happens in a longer, one-time session, for which a desktop screen is a welcome bonus but not required (BRIEF.md, Scale & Non-Functional Expectations: "mobile-first web for both roles... Desktop is a bonus for the pro's setup screens"). She checks Chairtime reactively throughout a working day rather than on a fixed schedule.

### What This User Does NOT Need

- Staff scheduling, multi-chair management, or any concept of "team" — the product is strictly single-operator (BRIEF.md, Constraints: "strictly single-operator for v1")
- Health or medical intake forms — massage and tattoo intake is explicitly out of scope for v1 (BRIEF.md, Constraints)
- Any exposure to card numbers or payment credentials — she never wants to see or store one (BRIEF.md, Constraints: "I never want to see or store a card number")
- Instagram integration beyond having a link to share — no DM automation, no feed posting (BRIEF.md, Ecosystem & Integrations: "No Instagram integration for v1")
- A native app or app-store install — she wants something that works the moment a client taps a link (BRIEF.md, Constraints: "no native apps, no app stores")

## Secondary Personas

### The Client

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section names the Client as a distinct role with its own goals, constraints ("must not face a signup wall"), and privacy boundary ("never visible to any other pro or client"), which differ entirely from the Pro's entitlements.

**Name:** Riley

**Description:** Riley saw a fresh set of lashes on Instagram and tapped the artist's bio link. Riley is not signing up for a "platform" — Riley wants to book one appointment with one person, quickly, from inside the Instagram in-app browser, without creating a password-protected account (BRIEF.md, The Experience; Target Users & Roles).

**Goals:**
- Book an appointment in under a minute without leaving the Instagram app experience (BRIEF.md, The Experience)
- Know exactly what the deposit rule is before paying anything (BRIEF.md, The Experience: "the deposit rule in plain words")
- Get a clear confirmation and a helpful reminder, and be able to reschedule or cancel without hunting for a phone number (BRIEF.md, The Experience; Target Users & Roles)
- See only their own upcoming and past bookings with this one pro — nothing more (BRIEF.md, Target Users & Roles)

**Pain Points:**
- Booking by DM means waiting for a reply and negotiating back and forth before a time is even confirmed (BRIEF.md, Problem Statement)
- No current lightweight way to prove or manage a booking without a full account (BRIEF.md, Open Questions: "Client identity")

**Behavioral Context:** Riley books from a phone, inside Instagram's in-app browser, in a single short session triggered by seeing the pro's work on their feed or story. Riley returns briefly around reminder time (to confirm or reschedule) and, occasionally, to book again with the same pro later.

**What This User Does NOT Need:** An account with a password; visibility into the pro's other clients or bookings; access to any other pro's booking page or client data (BRIEF.md, Target Users & Roles, Constraints: personal-data privacy).

### Platform Operator (Support)

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section explicitly names this as "Platform operator support access (not a product role)... a read-only support view of a pro's account," included "so downstream work accounts for it."

**Name:** N/A — this is the founder/operator function, not a persona to design experiences around.

**Description:** The founder, in an operational capacity, occasionally needs to look into a specific pro's account to help them resolve a problem (a missed payment, a confusing booking, a support ticket).

**Goals:** Diagnose and help resolve a pro's account issue without needing the pro to screen-share or describe every setting verbally.

**Pain Points:** N/A — this is an internal operational need, not a user pain point the product is designed to solve.

**Behavioral Context:** Occasional, on-demand use triggered by a support request; never a daily or scheduled workflow.

**What This User Does NOT Need:** Any client-facing capability, any ability to edit a pro's data, bookings, or payments, and no access beyond what is needed to view and diagnose (BRIEF.md, Target Users & Roles: "There is no client-facing access and nothing more. It is minimal admin access").

## Access Matrix

| Role / Persona | Service & Availability Setup | Booking & Payment | Client Records | Cancellation & No-Show Handling | Messaging & Consent | Subscription & Billing |
|---|---|---|---|---|---|---|
| The Pro (Talia) | Full | Full | Full | Full | Full | Full |
| The Client (Riley) | None | Own-only | None | Own-only | Own-only (their own consent) | None |
| Platform Operator (Support) | View | View | View (read-only, for troubleshooting only) | View | View | View |
