---
document_type: user-persona
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed (1 fix applied)
---

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
