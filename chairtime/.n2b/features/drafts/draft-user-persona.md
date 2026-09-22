---
document_type: user-persona
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-21
coherence_check: passed
---

# User Personas

## Persona Set Summary

This product serves two grounded product roles named in BRIEF.md's Target Users & Roles section: the Pro (primary persona, the business owner) and the Client (secondary persona, someone who books with that Pro). A third, narrow actor — Platform Operator support — is confirmed in the same brief section as read-only and explicitly "not a product role"; it is modeled only in the Access Matrix below, not as a full persona, because it has distinct entitlements the blueprint must document even though it never touches the product's actual user experience.

## Primary Persona

### Persona Name

Mara

### Description

Mara is an independent lash and brow artist working out of a home studio, one of the "huge number of these pros [who] went solo in the last few years and live on Instagram" (BRIEF.md, Business Context). She has no staff, no front desk, and no scheduling software budget to speak of — her diary today is a mix of a paper notebook, Google Calendar, and whatever a client last said in an Instagram DM. She is good at her craft and comfortable on her phone, but she is not a business-software person and has no patience for anything that feels built for a multi-chair salon.

### Goals

- Stop losing money to no-shows and last-minute cancellations without having to chase anyone for it
- Stop spending evenings negotiating appointment times over Instagram DM
- See, at a glance between clients, who is booked today, who has paid, and what is still owed
- Set her own prices, deposit rule, hours, buffer time, and cancellation policy once, and have the product enforce them automatically
- Look like a professional, dependable business to clients who found her on Instagram

### Pain Points

- Clients agree a time by DM, but nothing holds the slot, so double bookings happen and Mara only finds out when two people show up at once (BRIEF.md, Problem Statement)
- "Send me $20 on Venmo to hold your spot" deposits routinely never arrive, so the deposit doesn't actually protect the appointment (BRIEF.md, Problem Statement)
- When a client disputes a no-show charge, Mara has no record to point to — no proof of what was agreed or when (BRIEF.md, Problem Statement)
- Admin — confirming times, sending reminders — happens by hand, late at night, after a full day in the chair (BRIEF.md, Problem Statement)
- Existing tools are either sized and priced for multi-staff salons with setup Mara will never use, or free calendar links that cannot collect a deposit at all (BRIEF.md, Problem Statement)

### Behavioral Context

Mara reaches for the product on her phone, in short bursts, between clients — checking today's list, confirming a payment badge, or marking a no-show (BRIEF.md, The Experience). Less frequently, she sits down to adjust her services, prices, or policies. She books from home or her chair, not from a desk, and expects the product to work as well inside a normal mobile browser as any app she already uses.

### What This User Does NOT Need

- Multi-staff scheduling, chair assignment, or salon-wide management — the brief confirms this is strictly a single-operator product (BRIEF.md, Target Users & Roles)
- Any interface for handling or viewing raw card numbers — the processor owns that, and the brief is explicit the product's own code never sees or stores a card number (BRIEF.md, Ecosystem & Integrations)
- A native app or app-store presence — v1 is mobile-first web only (BRIEF.md, Constraints)
- Per-booking transaction pricing or a percentage cut — the brief states pros resent this and it is how they choose tools (BRIEF.md, Business Context)
- Enterprise-style reporting, staff permissions, or multi-location tooling — none of this exists for a solo operator

## Secondary Personas

### Taylor (the Client)

**Provenance:** [INFERRED from: brief passage "The Client. Someone who found the pro on Instagram and taps the bio link. Books, pays the deposit, reschedules or cancels within the policy window, and sees only their own upcoming and past bookings with that pro." — BRIEF.md directly establishes a second role with entitlements clearly distinct from the Pro's: Taylor may only ever act on their own bookings, never see Mara's client list, and needs no password-style account (BRIEF.md, Target Users & Roles).]

**Name:** Taylor

**Description:** Taylor found Mara through Instagram and wants an appointment. Taylor is not going to create an account, remember a password, or install anything — the brief is explicit that "the lightest workable identity" is the goal and there is "no signup wall" (BRIEF.md, Open Questions). Taylor books, pays a deposit, and moves on with their day.

**Goals:** Book a real, held appointment in under a minute without DMing anyone; know exactly what the deposit and cancellation rule are before paying; get reminded before the appointment; reschedule or cancel without a phone call if plans change.

**Pain Points:** Booking by DM means waiting for a reply and never knowing if a time is truly held; sending a deposit by Venmo feels informal and easy to dispute either way; no reminder means appointments get forgotten.

**Behavioral Context:** Taylor opens the link from inside Instagram's in-app browser (BRIEF.md, Ecosystem & Integrations) — almost never as a separate app or bookmarked site — and interacts only at two moments: booking, and responding to a reminder days later.

**What This User Does NOT Need:** An account, password, or profile to manage; visibility into any other client's bookings or Mara's day; any view of Mara's business settings, pricing rationale, or other clients.

### Platform Operator (Support Actor — Not a Persona)

**Provenance:** [INFERRED from: brief passage "Platform operator support (narrow actor, not a product role). The founder, as operator, needs a read-only look at a pro's setup and bookings to troubleshoot — never acting on the pro's behalf and never seeing more of client data than the pro's own screens show." — BRIEF.md explicitly confirms this actor exists and carries distinct, read-only entitlements, so the Access Matrix must model it even though the brief itself declines to call it a product role.]

**Name:** Operator (the founder, wearing a support hat)

**Description:** The founder, acting as platform operator, occasionally needs to see what a specific Pro sees — their setup and their bookings — in order to help when something goes wrong. This is a support capability, not a user-facing experience; there is no dashboard designed around this actor's own goals the way there is for Mara or Taylor.

**Goals:** Diagnose a reported problem (a missing confirmation, a sync issue, a disputed no-show) by seeing the same data the Pro sees, without altering it.

**Pain Points:** N/A — this is an internal support capability, not a persona with product-driven frustrations of their own.

**Behavioral Context:** Invoked only when a Pro reports an issue; read-only, time-boxed to the troubleshooting session, and scoped to exactly one Pro's data at a time.

**What This User Does NOT Need:** The ability to act on a Pro's behalf (create, edit, or cancel bookings, issue refunds, or change settings); visibility into any client data beyond what that Pro's own screens already show (BRIEF.md, Target Users & Roles); a dashboard of their own beyond the read-only support view.

## Access Matrix

| Role / Persona | Booking & Scheduling | Deposits & Payments | Client Records | Business Configuration | Operator Support Console |
|---|---|---|---|---|---|
| Mara (Pro) | Full | Full | Full (own clients only) | Full | None |
| Taylor (Client) | Own-only | Own-only | None | None | None |
| Operator (support actor) | View (read-only) | View (read-only) | View (read-only, no more than the Pro's own screens show) | View (read-only) | Full |
