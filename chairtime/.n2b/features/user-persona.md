---
document_type: user-persona
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-21
synthesis_check: passed (1 fix applied)
---

# User Personas

## Persona Set Summary

This product serves two grounded product roles named in BRIEF.md's Target Users & Roles section: the Pro (primary persona, the business owner) and the Client (secondary persona, someone who books with that Pro). A third, narrow actor — Platform Operator support — is confirmed in the same brief section as read-only and explicitly "not a product role"; it is modeled only in the Access Matrix below, not as a full persona, because it has distinct entitlements the blueprint must document even though it never touches the product's actual user experience. Market research corroborates this two-role (plus narrow support actor) shape directly: team/staff-role features are a market-wide pattern scoped specifically to multi-operator businesses (Vagaro, Booksy, Fresha), which is descriptive context about the wider category, not evidence this single-operator product needs an additional role [RESEARCH-INFORMED: source: market-research.md, Insights for This Product — "Team/staff-role features are a market-wide pattern scoped to multi-operator businesses, not evidence this product needs a role beyond Pro and Client," confidence: derived from Feature Landscape differentiators]. [INFERRED: carried from Visionary draft, enriched]

## Primary Persona

### Persona Name

Mara

### Description

Mara is an independent lash and brow artist working out of a home studio, one of the "huge number of these pros [who] went solo in the last few years and live on Instagram" (BRIEF.md, Business Context). She has no staff, no front desk, and no scheduling software budget to speak of — her diary today is a mix of a paper notebook, Google Calendar, and whatever a client last said in an Instagram DM. She is good at her craft and comfortable on her phone, but she is not a business-software person and has no patience for anything that feels built for a multi-chair salon. [RESEARCH-INFORMED: market research confirms solo/independent-first positioning is an active, recognized market segment rather than an underserved niche — GlossGenius explicitly markets to "solopreneurs" and theCut is purpose-built for barbers rather than adapted from salon software (source: market-research.md, GlossGenius and theCut profiles, confidence: MEDIUM) — reinforcing that Mara represents a real, already-served-but-imperfectly-served buyer, not a hypothetical one.]

### Goals

- Stop losing money to no-shows and last-minute cancellations without having to chase anyone for it
- Stop spending evenings negotiating appointment times over Instagram DM
- See, at a glance between clients, who is booked today, who has paid, and what is still owed
- Set her own prices, deposit rule, hours, buffer time, and cancellation policy once, and have the product enforce them automatically
- Look like a professional, dependable business to clients who found her on Instagram

<!-- [INFERRED: carried from Visionary draft] -->

### Pain Points

- Clients agree a time by DM, but nothing holds the slot, so double bookings happen and Mara only finds out when two people show up at once (BRIEF.md, Problem Statement)
- "Send me $20 on Venmo to hold your spot" deposits routinely never arrive, so the deposit doesn't actually protect the appointment (BRIEF.md, Problem Statement). [RESEARCH-INFORMED: this is not a minor or idiosyncratic annoyance — industry-wide data reports no-shows costing the US beauty industry an estimated $26 billion annually, with individual salons losing $1,500–$3,000 per month to unfilled appointments, and deposit-at-booking systems reported to cut no-show rates roughly fivefold versus none (source: market-research.md, Market Context — industry-statistics compilations, confidence: MEDIUM).]
- When a client disputes a no-show charge, Mara has no record to point to — no proof of what was agreed or when (BRIEF.md, Problem Statement). [RESEARCH-INFORMED: this specific gap — no concrete answer when a charge is disputed — is corroborated across the wider market: every profiled competitor with reviewable enforcement evidence has at least one documented case of a deposit or no-show charge failing silently or without notification (source: market-research.md, Common Complaint Themes, confidence: HIGH), meaning Mara's current tools would not close this gap even if she adopted one of them as-is.]
- Admin — confirming times, sending reminders — happens by hand, late at night, after a full day in the chair (BRIEF.md, Problem Statement)
- Existing tools are either sized and priced for multi-staff salons with setup Mara will never use, or free calendar links that cannot collect a deposit at all (BRIEF.md, Problem Statement)

### Behavioral Context

Mara reaches for the product on her phone, in short bursts, between clients — checking today's list, confirming a payment badge, or marking a no-show (BRIEF.md, The Experience). Less frequently, she sits down to adjust her services, prices, or policies. She books from home or her chair, not from a desk, and expects the product to work as well inside a normal mobile browser as any app she already uses.

<!-- [INFERRED: carried from Visionary draft] -->

### What This User Does NOT Need

- Multi-staff scheduling, chair assignment, or salon-wide management — the brief confirms this is strictly a single-operator product (BRIEF.md, Target Users & Roles)
- Any interface for handling or viewing raw card numbers — the processor owns that, and the brief is explicit the product's own code never sees or stores a card number (BRIEF.md, Ecosystem & Integrations)
- A native app or app-store presence — v1 is mobile-first web only (BRIEF.md, Constraints)
- Per-booking transaction pricing or a percentage cut — the brief states pros resent this and it is how they choose tools (BRIEF.md, Business Context). [RESEARCH-INFORMED: market research confirms this resentment is real and specific, not assumed — the per-new-client marketplace fee charged by StyleSeat, Booksy (opt-in), and Fresha is the space's most consistently documented professional-side complaint, especially when repeat or referred clients are misclassified as "new" and charged again (source: market-research.md, StyleSeat and Fresha profiles; Common Complaint Themes, confidence: HIGH).]
- Enterprise-style reporting, staff permissions, or multi-location tooling — none of this exists for a solo operator
- Consumer-facing client-discovery marketplace exposure or AI-assisted after-hours inquiry handling — differentiators some competitors offer (theCut, StyleSeat, Booksy, Fresha), but Mara's whole point of adoption is replacing Instagram-DM discovery with her own link, not adding a second discovery channel that comes bundled with the fee model she is explicitly avoiding [RESEARCH-INFORMED: source: market-research.md, Feature Landscape — Differentiators, confidence: MEDIUM; see scope-boundaries.md for the corresponding exclusion].

## Secondary Personas

### Taylor (the Client)

**Provenance:** [INFERRED from: brief passage "The Client. Someone who found the pro on Instagram and taps the bio link. Books, pays the deposit, reschedules or cancels within the policy window, and sees only their own upcoming and past bookings with that pro." — BRIEF.md directly establishes a second role with entitlements clearly distinct from the Pro's: Taylor may only ever act on their own bookings, never see Mara's client list, and needs no password-style account (BRIEF.md, Target Users & Roles).]

**Name:** Taylor

**Description:** Taylor found Mara through Instagram and wants an appointment. Taylor is not going to create an account, remember a password, or install anything — the brief is explicit that "the lightest workable identity" is the goal and there is "no signup wall" (BRIEF.md, Open Questions). Taylor books, pays a deposit, and moves on with their day.

**Goals:** Book a real, held appointment in under a minute without DMing anyone; know exactly what the deposit and cancellation rule are before paying; get reminded before the appointment; reschedule or cancel without a phone call if plans change; be able to stop receiving texts at any time without having to ask Mara to do it for her [AUDIT-ADDED: 3 -- Entity Coverage Verification's inverse check found no feature specified how a client withdraws messaging consent; product-features.md FEAT-16 now specifies a standard STOP-reply mechanism, which this goal reflects].

**Pain Points:** Booking by DM means waiting for a reply and never knowing if a time is truly held; sending a deposit by Venmo feels informal and easy to dispute either way; no reminder means appointments get forgotten.

**Behavioral Context:** Taylor opens the link from inside Instagram's in-app browser (BRIEF.md, Ecosystem & Integrations) — almost never as a separate app or bookmarked site — and interacts only at two moments: booking, and responding to a reminder days later.

**What This User Does NOT Need:** An account, password, or profile to manage; visibility into any other client's bookings or Mara's day; any view of Mara's business settings, pricing rationale, or other clients.

### Platform Operator (Support Actor — Not a Persona)

**Provenance:** [INFERRED from: brief passage "Platform operator support (narrow actor, not a product role). The founder, as operator, needs a read-only look at a pro's setup and bookings to troubleshoot — never acting on the pro's behalf and never seeing more of client data than the pro's own screens show." — BRIEF.md explicitly confirms this actor exists and carries distinct, read-only entitlements, so the Access Matrix must model it even though the brief itself declines to call it a product role.]

**Name:** Operator (the founder, wearing a support hat)

**Description:** The founder, acting as platform operator, occasionally needs to see what a specific Pro sees — their setup and their bookings — in order to help when something goes wrong. This is a support capability, not a user-facing experience; there is no dashboard designed around this actor's own goals the way there is for Mara or Taylor. [RESEARCH-INFORMED: support responsiveness is a documented, cross-competitor weak point — slow response times, multi-agent hand-offs, and chatbot-first support drew complaints across Booksy, theCut, and Fresha reviews (source: market-research.md, Common Complaint Themes, confidence: HIGH). This reinforces the value of the founder's own direct, human, read-only troubleshooting posture at this stage, rather than justifying any change to the actor's narrow, non-acting scope.]

**Goals:** Diagnose a reported problem (a missing confirmation, a sync issue, a disputed no-show) by seeing the same data the Pro sees, without altering it.

**Pain Points:** N/A — this is an internal support capability, not a persona with product-driven frustrations of their own.

**Behavioral Context:** Invoked only when a Pro reports an issue; read-only, time-boxed to the troubleshooting session, and scoped to exactly one Pro's data at a time. Every lookup the Operator performs is itself logged (which Pro, which booking, when) so the practice remains accountable to its own promise [AUDIT-ADDED: 4 -- Cross-Cutting Concerns Verification (Audit Logging) found this actor's own access needed a record; see product-features.md FEAT-21].

**What This User Does NOT Need:** The ability to act on a Pro's behalf (create, edit, or cancel bookings, issue refunds, or change settings); visibility into any client data beyond what that Pro's own screens already show (BRIEF.md, Target Users & Roles); a dashboard of their own beyond the read-only support view.

## Access Matrix

| Role / Persona | Booking & Scheduling | Deposits & Payments | Client Records | Business Configuration | Operator Support Console |
|---|---|---|---|---|---|
| Mara (Pro) | Full | Full | Full (own clients only) | Full | None |
| Taylor (Client) | Own-only | Own-only | None | None | None |
| Operator (support actor) | View (read-only) | View (read-only) | View (read-only, no more than the Pro's own screens show) | View (read-only) | Full |

<!-- Access Matrix audit (completeness-audit.md): confirmed to cover all three actors and every major capability group of the reconciled feature set, including the audit-added capabilities. Pro-initiated reschedule/cancellation (FEAT-07) and STOP-based consent withdrawal (FEAT-16) fall under "Booking & Scheduling" for the acting persona (Mara Full / Taylor Own-only respectively); Operator action logging (FEAT-21) falls under "Operator Support Console" (Full for the Operator, None for Mara and Taylor, consistent with existing rows). No new column is required; every feature's Access field agrees with this matrix. [AUDIT-ADDED: 4 -- Access Matrix audit performed per completeness-audit.md instructions] -->
</content>
