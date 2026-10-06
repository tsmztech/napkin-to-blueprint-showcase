# Part C — Scope & Assumptions

This part records what Chairtime deliberately does not do, the assumptions and constraints the specifications rest on, and the open items the team still owns.

## What Is Out of Scope


# Scope Boundaries

## In-Scope Summary

This product is a mobile-first booking link for one independent beauty or wellness professional, letting a client pick a service and a genuinely free time, pay a card deposit that lands directly with the pro, and receive automatic reminders — with a no-show deposit kept automatically under the pro's own cancellation policy. Core features (FEAT-01 through FEAT-12, plus the audit-added FEAT-28 Payout Account Connection & Payout Visibility and FEAT-30 Pro Booking Management) close that entire loop end to end: setup, availability, calendar sync, booking, payment, payout, messaging, cancellation policy enforcement, pro-side changes, and no-show handling. Important features (FEAT-13 through FEAT-19, plus the audit-added FEAT-27 Pro Profile & Booking Page Settings and FEAT-29 Pro Sign-In & Account Lifecycle) add the record-keeping, onboarding, profile, sign-in, consent, billing, and support machinery a real single-operator business needs from day one. Nice-to-Have features (FEAT-20 through FEAT-26) answer the brief's own open questions — waitlist, recurring appointments, in-app balance payment, tipping — plus later polish (client search, insights, WhatsApp). The product serves exactly two product roles (the Pro and the Client) plus one narrow, non-product support role. [MODIFIED: in-scope summary updated to include the four audit-added features]

## Explicit Exclusions

### User Scope Exclusions

- **ID:** SC-01
- **Multi-staff or multi-chair salon accounts** — BRIEF.md's Constraints state the product is "strictly single-operator for v1... probably forever for this product"; a pro with staff or a multi-chair salon is a different product. [RESEARCH-INFORMED: payroll, multi-staff management and per-additional-user pricing appear in four of five profiled competitors, confirming this is the salon-scale pattern the brief deliberately avoids, from the market-research Feature Comparison Matrix]
- **ID:** SC-02
- **Any administrative, manager, or staff role beyond the founder's own read-only support access** — the persona set in user-persona.md confirms exactly two product roles plus one narrow support function ("no other roles," BRIEF.md, Target Users & Roles); no additional role is grounded in the brief.
- **ID:** SC-03
- **Cross-pro or cross-client visibility** — BRIEF.md's Constraints are explicit that a client's data is "never visible to any other pro or client"; there is no shared or comparative view across accounts of any kind.
- **ID:** SC-04
- **Client accounts with passwords, or a single client profile shared across several pros** — BRIEF.md's Target Users & Roles says clients "must not face a signup wall or need a password-style account"; a client's identity is their phone number with one pro only, so booking with a second pro creates a separate, unconnected record. [AUDIT-EXCLUDED: 4 -- Roles and Sharing concern: a cross-pro client profile would share client data between pros, contradicting BRIEF.md's privacy constraint]
- **ID:** SC-05
- **Support staff acting on a Pro's behalf** — BRIEF.md limits the operator to "a read-only support view"; support cannot edit, refund, sign in as a Pro, or change a Pro's sign-in details, so a Pro who loses access to both their sign-in email and phone must regain one of them to recover. [AUDIT-EXCLUDED: 4 -- Security and Privacy Posture concern: operator-performed account changes would exceed the brief's minimal-admin boundary]

### Feature Scope Exclusions

- **ID:** SC-06
- **Instagram integration beyond serving as the destination for the bio link** — BRIEF.md's Ecosystem & Integrations states plainly: "No Instagram integration for v1." Chairtime is what the link points to, nothing more.
- **ID:** SC-07
- **Native mobile apps or app-store distribution** — BRIEF.md's Constraints specify "mobile-first web only for v1; no native apps, no app stores," matching where users already are (Instagram's in-app browser).
- **ID:** SC-08
- **Health or medical intake forms, and custom intake questionnaires generally** — BRIEF.md's Constraints explicitly exclude this: "massage/tattoo intake forms are out of scope for v1," and confirm no health-data regime applies; the only free text a client adds is a short optional note to the Pro, with a hint not to include medical information. [AUDIT-EXCLUDED: 2 -- custom intake questions are a differentiator in one competitor (Booksy); adding them would invite health data the brief keeps out of scope]
- **ID:** SC-09
- **Importing data from prior tools (DM history, paper diaries, spreadsheets)** — pros switching to Chairtime are leaving an informal, unstructured process (Instagram DMs, a paper notebook, ad hoc Venmo requests per BRIEF.md's Problem Statement); there is no structured source worth building an import path for at this scale.
- **ID:** SC-10
- **Multi-language or translated content** — BRIEF.md's Scale & Non-Functional Expectations names the US, UK, Canada, and Australia as the near-term geography, all English-language markets; no translation need is stated.
- **ID:** SC-11
- **Handling or storing card data within the product itself** — BRIEF.md's Constraints state directly: "card data is never stored or handled by the founder's code; the payment processor owns it." This is a hard boundary, not a feature choice.
- **ID:** SC-12
- **Social features, public reviews, marketplace discovery, or discoverability profiles** — the brief's entire discovery model is the pro's own Instagram presence (BRIEF.md, Ecosystem & Integrations); building a competing discovery or social layer inside Chairtime would change the product's fundamental nature. [RESEARCH-INFORMED: the two competitors with marketplace discovery fund it with a one-off 20–30% new-client commission that professionals widely resent, which BRIEF.md's "absolutely no per-booking cut" rules out, from StyleSeat and Fresha pricing-guide aggregators (HIGH confidence)]
- **ID:** SC-13
- **Card-on-file cancellation fees charged after booking** — the product protects the pro with a deposit paid up front (BRIEF.md, Vision), and charging a client's card later would require keeping a card on file for future charges. [AUDIT-EXCLUDED: 2 -- Booksy offers a card-on-file fee mode alongside deposits; excluded because the brief specifies deposits, and after-the-fact charges are exactly the disputed-charge pattern clients complain about (MEDIUM confidence)]
- **ID:** SC-14
- **Dynamic, demand-based, or time-of-day pricing** — BRIEF.md describes one price per service; varying prices by demand or time adds setup the solo pro did not ask for. [AUDIT-EXCLUDED: 2 -- "smart pricing" is an optional differentiator in one competitor, and the inability to vary price by day is a complaint about another (MEDIUM confidence); not aligned with the brief's set-up-once vision, so revisit only if pros ask for it]
- **ID:** SC-15
- **Marketing or promotional text campaigns** — clients consent to booking-related texts only (BRIEF.md, Constraints: "reminders must respect that consent"), and Chairtime sends nothing beyond confirmations, reminders, change notices, and access links. [AUDIT-EXCLUDED: 2 -- text marketing is an add-on in two competitors; excluded because it would stretch the client's booking-specific consent and add the à-la-carte complexity pros complain about]
- **ID:** SC-16
- **Card-reader hardware or in-person payment taking** — at MVP, the balance after the deposit is settled in person between the pro and the client, outside the platform, by whatever means the pro already uses; the pro simply records the booking as completed. From v1, clients may optionally pay the balance in-app (FEAT-22). [AUDIT-EXCLUDED: 1 -- value-flow walk: the balance segment at MVP is deliberately owned by the pro, off-platform; hardware billing disputes are also a reported complaint about one competitor (LOW confidence)]
- **ID:** SC-17
- **Chairtime deciding disputes between a pro and a client** — the product provides the trustworthy record (FEAT-16) and the pro can refund as goodwill (FEAT-30), but Chairtime never rules on who is right; a client who disagrees can contest the charge with their card issuer, handled through the payment processor. [AUDIT-EXCLUDED: 1 -- counterpart symmetry: dispute outcomes are owned by the pro and the payment processor's dispute process, not by the platform]
- **ID:** SC-18
- **Partial refunds and tiered cancellation schedules** — BRIEF.md's Business Context defines a binary rule (refund outside the window, kept inside it or on a no-show); partial percentages by timing are not offered in v1. [AUDIT-EXCLUDED: 1 -- value-flow walk: keeping the rule binary keeps every deposit outcome predictable for both parties]

### Scale Expectations

- **ID:** SC-19
- **Volume:** a few hundred pros in year one, each with roughly 100–500 clients and 20–40 bookings a week, honored from MVP onward — matching BRIEF.md's Scale & Non-Functional Expectations exactly; no separate scale phase-in is needed at this volume.
- **ID:** SC-20
- **Geography phase-in:** the product launches US-first at MVP; timezone and currency are treated as per-account configuration from day one (never hard-coded) so that expansion to the UK, Canada, and Australia at v1/Later requires no structural rework — per BRIEF.md's stated expansion intent. Each pro's payout account must be in the same country as their Chairtime account.
- **ID:** SC-21
- **Correctness over uptime numbers:** BRIEF.md states no specific uptime target is required, but that "booking and payment must be correct, always" and the product must "never silently double-book or lose a deposit." This bar applies at every scale from MVP onward; it is a correctness expectation, not a boundary that phases in.
- **ID:** SC-22
- **History depth:** a pro's full booking, client and deposit history is kept for as long as their account exists, so multi-year history stays available for dispute evidence and insights; after a client deletion or account closure, only de-identified financial records required by law are retained. [AUDIT-ADDED: 3 -- entity coverage: retention of Booking, Client and Deposit Transaction history was unstated]

## Deferral Notes

- **Waitlist for Cancelled Slots** — Target phase: v1. Deferred rather than rejected; it depends on Client-Initiated Cancel/Reschedule already existing and proven, and the core booking loop works without it. Worth including once cancellations are common enough to make a waitlist meaningful. [RESEARCH-INFORMED: the closest solo-focused competitor offers waitlists as a business tool (MEDIUM confidence)]
- **Recurring/Standing Appointments** — Target phase: v1. BRIEF.md's Open Questions asks directly whether this is v1 or later; deferred so the single-booking core loop is proven reliable first, given the added complexity to the availability engine. At MVP, a pro can already rebook a regular one visit at a time through Pro Booking Management.
- **In-App Balance Payment** — Target phase: v1. BRIEF.md's Open Questions leaves this open; the default in-person balance flow works without it, so it is deferred as a low-risk enhancement once deposit payment is proven in production.
- **Tipping at Checkout** — Target phase: Later. BRIEF.md's Open Questions asks where tipping fits "if anywhere"; deferred because it depends on In-App Balance Payment existing first and has no bearing on the core no-show problem.
- **Client List Search & Filter** — Target phase: v1. A new pro starts with very few clients, so this becomes valuable only as a pro's client base grows toward the brief's stated 100–500 range.
- **Booking & Revenue Insights** — Target phase: v1. Deferred until pros have enough booking history for a summary to be meaningful; not required for the core loop to deliver value.
- **WhatsApp Reminders** — Target phase: Later. BRIEF.md's Ecosystem & Integrations states this directly: "WhatsApp is a nice-to-have later, not v1."


## Assumptions, Constraints & Expectations


# Assumptions and Constraints

## Product Assumptions

### User Environment

- **ID:** ASMP-01
- **We assume clients access the booking page primarily from inside Instagram's in-app browser on a phone.** Invalidated if: usage data shows a majority of client sessions arrive from outside Instagram (direct link shares, other social platforms), which would change which in-app-browser accommodations matter most.
- **ID:** ASMP-02
- **We assume pros run their day-to-day schedule from a phone, reserving desktop for one-time setup only.** Invalidated if: usage data shows pros regularly running their live daily schedule from a desktop browser during working hours.
- **ID:** ASMP-03
- **We assume both pros and clients have reliable, if intermittent, mobile data access at the moments they use the product.** Invalidated if: a meaningful share of target pros or clients regularly operate in low- or no-connectivity settings during booking or check-in, which would require rethinking the product's deliberate online-only correctness stance.

### User Behavior

- **ID:** ASMP-04
- **We assume clients are willing to pay a card deposit to an individual businessperson they found on Instagram, without an established brand behind the request.** Invalidated if: booking-funnel data shows a large share of clients abandon specifically at the deposit-payment step, citing distrust of the individual pro rather than price or friction.
- **ID:** ASMP-05
- **We assume pros will set their own cancellation policy in a way that prevents most no-shows without generating frequent client disputes.** Invalidated if: dispute rates (per the Policy Clarity at Booking metric) stay persistently high across pros, suggesting policies are being set in ways clients don't understand or accept at booking time.
- **ID:** ASMP-06
- **We assume a pro checks Chairtime in short, reactive bursts between clients rather than in one planning session per day.** Invalidated if: usage data shows pros primarily reviewing their schedule once per day rather than throughout the day.
- **ID:** ASMP-07
- **We assume most clients will opt in to text messages when they book.** Invalidated if: fewer than half of clients opt in to texts across pros, which would make email the main reminder channel and weaken the one-tap reminder experience the Reminder Response Rate metric depends on. [AUDIT-ADDED: 1 -- journey walk: the email fallback added to the booking flow is only a fallback if most clients choose texts]
- **ID:** ASMP-08
- **We assume pros will complete the payment processor's identity and bank verification during setup without help.** Invalidated if: more than 1 in 5 pros who reach the payout step have not finished verification within a week, which would make payout setup the main barrier to a live booking link. [AUDIT-ADDED: 1 -- value-flow walk: the audit-added payout-account step now gates the booking link going live]

### Product Context

- **ID:** ASMP-09
- **We assume this product remains a standalone booking-and-deposit tool, not a broader salon or business-management suite.** Invalidated if: founder or pro feedback reveals sustained demand for adjacent capabilities (inventory, staff payroll, multi-service business management) that only make sense for a multi-person business — which would contradict the strictly single-operator positioning.
- **ID:** ASMP-10
- **We assume Instagram remains the pros' primary client-discovery channel throughout the period this blueprint covers.** Invalidated if: target pros shift their client-discovery activity to a different platform in large numbers, which would change where the booking link needs to live and how it's shared.
- **ID:** ASMP-11
- **We assume the MVP scope — including two-way sync with both Google and Apple calendars and the audit-added payout, sign-in, profile and pro-side booking features — can reach a first paying pro within about three months for a solo founder building with AI coding tools.** Invalidated if: by the end of month two the core loop (book, pay deposit, remind, cancel/refund, no-show) does not yet work end to end, which would call for re-phasing lower-risk MVP items with the founder rather than cutting the correctness bar. [AUDIT-ADDED: 4 -- BRIEF.md's Constraints set a three-month timeline that the final 23-feature MVP must be checked against]
- **ID:** ASMP-12
- **We assume a solo pro's clients are willing to pay each deposit fresh rather than keep a card on file with the pro.** Invalidated if: a meaningful share of repeat clients abandon rebooking at the deposit step, citing re-entering card details, which would argue for a processor-held saved-card option later. [RESEARCH-INFORMED: competitors lean on card-on-file mechanisms, but difficulty removing stored client cards is a frequently mentioned complaint about one of them (MEDIUM confidence), supporting BRIEF.md's no-stored-card posture]

## Product Constraints

- **ID:** ASMP-13
- **Single-operator only** — BRIEF.md's Constraints state this is deliberate and permanent ("probably forever for this product"), not a phased limitation to be lifted later; multi-staff and multi-chair scheduling are a different product entirely.
- **ID:** ASMP-14
- **Flat monthly subscription, no per-booking fee** — a deliberate business-model constraint per BRIEF.md's Business Context: pros "resent" per-booking cuts, and it is described as "how they choose tools." This shapes pricing and billing design, not just a default.
- **ID:** ASMP-15
- **No card data ever touches the product's own code** — a deliberate security and regulatory constraint per BRIEF.md's Constraints ("I never want to see or store a card number"); all card handling is delegated entirely to the payment-processing capability.
- **ID:** ASMP-16
- **Mobile-first web only, no native apps** — a deliberate platform constraint per BRIEF.md's Constraints, matching where users already are (phone, Instagram in-app browser) and the founder's speed-to-launch goal.
- **ID:** ASMP-17
- **The product enforces whatever cancellation policy the pro sets; it offers a common default as a starting point during setup but never overrides a policy the pro has chosen** — a deliberate constraint keeping business judgment with the pro rather than the platform, consistent with the brief's framing of the policy as the pro's own rule that clients agree to. [MODIFIED: "never recommends" softened to "offers a common default as a starting point" because the onboarding wizard (FEAT-15) proposes a default window, which the draft constraint contradicted -- synthesis check fix]
- **ID:** ASMP-18
- **Money never rests with the platform** — client deposits (and, from v1, balances and tips) go from the client's card straight to the pro's own payout account through the payment processor; Chairtime takes no cut and never holds or routes funds itself. A deliberate business-model and trust constraint per BRIEF.md's Business Context ("the platform takes no cut of any of it"). [AUDIT-ADDED: 1 -- value-flow walk: the path of every unit of money needed to be stated as a product rule]
- **ID:** ASMP-19
- **Deposit outcomes are binary and symmetric** — outside the window a client's cancellation is refunded in full, inside it (or on a no-show) the deposit is kept, and any cancellation made by the pro is always refunded in full. A deliberate fairness constraint extending BRIEF.md's Business Context to the case where the pro is the party who cancels. [AUDIT-ADDED: 1 -- counterpart symmetry]
- **ID:** ASMP-20
- **Support access stays read-only and visible** — the operator can look but never change anything, and every look is recorded in the pro's own account activity. A deliberate constraint per BRIEF.md's Target Users & Roles ("minimal admin access"). [AUDIT-ADDED: 4 -- Audit Logging concern]

## Non-Functional Expectations

- **ID:** ASMP-21
- **Responsiveness: available slots appear within roughly one second of a service selection, and a full booking (selection through paid confirmation) completes in under one minute.** — Basis: BRIEF.md's Vision states the one-minute booking benchmark directly, and its Scale & Non-Functional Expectations section makes correctness and speed the product's defining quality bar.
- **ID:** ASMP-22
- **Data volume and growth: a few hundred pros in year one, each with roughly 100–500 clients and 20–40 bookings a week, with the product staying equally responsive as pros accumulate history over multiple years.** — Basis: BRIEF.md's Scale & Non-Functional Expectations, stated directly.
- **ID:** ASMP-23
- **Privacy posture: a client's data is visible only to their own pro and to themselves; a pro can permanently delete a client's record on request.** — Basis: BRIEF.md's Privacy and Constraints sections, stated directly.
- **ID:** ASMP-24
- **Compliance: US SMS-consent rules apply to all client texting (explicit opt-in captured at booking, honored immediately on opt-out); no health-data regime applies, since intake and clinical data are explicitly out of scope.** — Basis: BRIEF.md's Constraints, "Regulated-domain confirmation" section, stated directly.
- **ID:** ASMP-25
- **Geography and localization: timezone and currency are per-account configuration from day one, never hard-coded, so expansion beyond the US requires no structural rework.** — Basis: BRIEF.md's Scale & Non-Functional Expectations: "the UK, Canada and Australia are the obvious next markets, so timezone and currency must not be hard-coded."
- **ID:** ASMP-26
- **Reliability is expressed as a correctness bar, not a numeric uptime target: the product must never silently double-book a slot or lose a deposit.** — Basis: BRIEF.md's Scale & Non-Functional Expectations states directly that "no specific uptime number was given" and frames correctness, not uptime percentage, as the requirement. [RESEARCH-INFORMED: glitches and crashes in the core scheduling workflow are reported for two competitors at comparable booking volumes (MEDIUM confidence), so correctness under everyday use is a documented market gap]
- **ID:** ASMP-27
- **Offline and loading posture: anything that books, pays, cancels, refunds or marks a no-show needs a live connection and says so plainly when it is missing; the pro's most recently loaded schedule, client list and money list stay readable offline; every screen that waits shows an in-place indicator rather than a blank page, and nothing appears tappable before real data has loaded.** — Basis: BRIEF.md's correctness bar ("never silently double-book or lose a deposit") and the decomposition checklist's Offline and Loading items; decided product-wide so every feature's States field follows one rule. [AUDIT-ADDED: 4 -- Offline/Degraded and Loading concerns needed a product-level decision]
- **ID:** ASMP-28
- **Accessibility baseline: every client and pro screen is readable and fully operable at phone width inside a social-media in-app browser, with text that scales, sufficient contrast, controls large enough to tap reliably, and full use by screen-reader users; nothing relies on color alone (for example, the paid badge also carries a word).** — Basis: BRIEF.md's Scale & Non-Functional Expectations (mobile-first, Instagram in-app browser) and the decomposition checklist's Accessibility item. [AUDIT-ADDED: 4 -- Accessibility baseline was not decided in the draft]
- **ID:** ASMP-29
- **Messaging timing: automatic reminders reach clients only during reasonable daytime hours (roughly 8am–9pm in the pro's timezone), and confirmations arrive within about a minute of payment.** — Basis: BRIEF.md's Constraints ("reminders must respect that consent," citing US texting rules) and its Vision ("a confirmation text lands immediately"). [AUDIT-ADDED: 4 -- Compliance concern]
- **ID:** ASMP-30
- **Account protection: a pro's account, which holds every client's contact details, is protected by a one-time-code sign-in with new-device alerts; client access links are short-lived and open only that client's bookings with that one pro.** — Basis: BRIEF.md's Privacy section ("a client's data is visible only to their pro") and the decomposition checklist's Security and Privacy Posture item. [AUDIT-ADDED: 4 -- Security and Privacy Posture concern]

## Dependencies

- **ID:** ASMP-31
- **Payment-processing capability** — required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription. Without it, the product has no way to collect money or generate revenue at all; BRIEF.md's Constraints require that this capability, not the product's own code, owns all card data. [MODIFIED: connected payout accounts, identity verification, refunds and dispute notifications named explicitly after the value-flow audit]
- **ID:** ASMP-32
- **Transactional text-messaging capability, with email as a fallback channel** — required to send booking confirmations and pre-appointment reminders. Without it, the product cannot deliver the automatic-reminder promise that replaces the pro's manual texting habit; BRIEF.md's Ecosystem & Integrations names texting as the v1 channel with email as an acceptable fallback.
- **ID:** ASMP-33
- **Calendar-sync capability (reading and writing to a pro's personal calendar)** — required for the two-way sync described in BRIEF.md's Ecosystem & Integrations. Without it, the availability engine cannot account for a pro's real-world commitments outside Chairtime, directly threatening the "never double-book" correctness bar.
- **ID:** ASMP-34
- **A searchable, per-pro record store for services, bookings, clients, and payment outcomes** — required for the product to function at all across sessions; without persistent, per-account data, nothing booked, paid, or configured could be relied on the next time either the pro or the client returns.
- **ID:** ASMP-35
- **File storage capability for pro profile photos** — required for the photo shown on the booking page (FEAT-27). Without it, the booking page shows the pro's name only; the booking loop itself is unaffected. [AUDIT-ADDED: 3 -- the profile photo captured by the audit-added Pro Profile & Booking Page Settings needs somewhere to live]


## Open Items for the Team

These are the brief's open questions — unresolved by design; the team owns them.

- **Balance payment:** should the balance after the deposit be payable in the app, or stay in person (cash / card reader at the chair)?
- **Waitlist:** should a pro be able to offer a waitlist for slots that open up from cancellations?
- **Tipping:** where does tipping fit, if anywhere?
- **Client identity:** what is the lightest workable identity for clients so they can manage their own bookings without a signup wall? A phone number plus a magic link, or something else?
- **Recurring appointments:** are recurring or standing appointments ("every 3 weeks") v1 or later?
- **Subscription price point:** the model is a flat monthly fee "cheap enough that a single saved no-show pays for the month", but the actual price was not stated.
- **Availability target:** there is no stated uptime number; the requirement is correctness ("never silently double-book, never lose a deposit"). Downstream should decide what availability that implies.


