---
document_type: assumptions-constraints
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# Assumptions and Constraints

## Product Assumptions

### User Environment

- **ID:** ASMP-01
- **We assume most household members access Plateful primarily on a smartphone, including inside a supermarket with variable signal.** Invalidated if: usage data shows a majority of grocery-list and plan interactions happen on desktop rather than mobile.
- **ID:** ASMP-02
- **We assume the organiser is willing to use a larger screen (laptop or tablet) at least once for the initial, more detailed household setup.** Invalidated if: setup completion data shows setup is overwhelmingly abandoned or never attempted on larger screens, indicating mobile-only setup must be the primary path instead.
- **ID:** ASMP-03
- **We assume households have reliable connectivity at home to receive and review the weekly plan, even if the shopping trip itself happens somewhere with weak signal.** Invalidated if: a meaningful share of target households report unreliable home internet access.

### User Behavior

- **ID:** ASMP-04
- **We assume the organiser reviews and actively engages with the AI-generated plan each week rather than ignoring it.** Invalidated if: usage data shows a large share of generated plans are never opened.
- **ID:** ASMP-05
- **We assume other adult household members actively use the shared grocery list themselves, rather than the organiser doing all the shopping and list management alone.** Invalidated if: list activity (ticks, manual adds) is concentrated almost entirely on the organiser's account across most households.
- **ID:** ASMP-06
- **We assume households are willing and able to enter accurate allergy and dietary-rule information during setup.** Invalidated if: support requests or safety reports reveal a pattern of incomplete or incorrect allergy data being entered at setup time.

### Product Context

- **ID:** ASMP-07
- **We assume Plateful can deliver its core value as a standalone product, without needing deep integration into an existing grocery-ordering or meal-kit ecosystem at launch.** Invalidated if: early adoption stalls specifically because households want online ordering integration (FEAT-20) before they will commit to the product.
- **ID:** ASMP-08
- **We assume growth will come primarily from household-to-household invitations rather than paid acquisition, at least in the first year.** Invalidated if: invitation-driven signups remain a small minority of new paying households after several months of operation.

## Product Constraints

- **ID:** ASMP-09
- **Allergies and religious rules are hard filters enforced by the app itself, never left to AI judgment alone.** BRIEF.md's Constraints section states the AI "is never the last line of defence" — this is a deliberate safety architecture choice, not a feature that could be relaxed for convenience.
- **ID:** ASMP-10
- **No advertising and no sale or third-party use of family data, ever.** BRIEF.md's Business Context names this directly as a deliberate trust-building positioning constraint, not merely a launch-time omission that might change with monetization pressure.
- **ID:** ASMP-11
- **Responsive web app only in v1, with no native apps and no app stores.** BRIEF.md's Constraints section states this is a deliberate solo-founder, ship-fast choice.
- **ID:** ASMP-12
- **One household per account in v1.** BRIEF.md's Target Users & Roles section states this directly as a deliberate scope constraint that keeps the account and access model simple at launch.
- **ID:** ASMP-13
- **No medical or diet advice is given by the product.** BRIEF.md's Constraints section states this explicitly, positioning Plateful as a planning tool rather than a health or medical product.

## Non-Functional Expectations

- **ID:** ASMP-14
- **Responsiveness: the shared grocery list feels instant when items are ticked or added, and a meal swap completes and reflects in the list within seconds.** Basis: BRIEF.md's Scale & Non-Functional Expectations section states the list "must feel instant," and the Vision states swaps update the list "instantly."
- **ID:** ASMP-15
- **Data volume and growth: several thousand households in the first year, 2-6 members each, with the product staying equally responsive as that base grows.** Basis: BRIEF.md's Scale & Non-Functional Expectations, Order of magnitude.
- **ID:** ASMP-16
- **Resilience: the shared grocery list keeps working, with changes queued and synced later, when used in a supermarket with poor or no signal.** Basis: BRIEF.md's Scale & Non-Functional Expectations, Performance/availability.
- **ID:** ASMP-17
- **Privacy posture: children's data is minimal, parent-controlled, and used only for the household's own plan; no data is ever sold or used for advertising to any household member.** Basis: BRIEF.md's Privacy section under Scale & Non-Functional Expectations and Constraints.
- **ID:** ASMP-18
- **Compliance: because the product processes children's data, it is treated as subject to children's-privacy-class protections (minimal collection, parental control, no behavioral advertising to minors) alongside general personal-data protection for all household members; no medical-data regime applies, since the product gives no medical or diet advice.** Basis: domain reasoning from decomposition-checklists.md's Compliance section, grounded in BRIEF.md's stated children's-privacy sensitivity and its explicit "no medical or diet advice" positioning.
- **ID:** ASMP-19
- **Localization: measurement units, currency, and supermarket aisle names are configurable per household to support both US and UK households from launch.** Basis: BRIEF.md's Scale & Non-Functional Expectations, Geography.

## Dependencies

- **ID:** ASMP-20
- **AI text/plan-generation capability** — Required to generate the weekly dinner plan and swap alternatives; without it, planning would revert to a fully manual, unassisted experience, and the product's core promise would not exist.
- **ID:** ASMP-21
- **Device-notification delivery capability** — Required for the weekly "plan ready" notification and the daily "tonight's dinner" nudge; without it, both degrade to in-app-only discovery, weakening the product's proactive rhythm.
- **ID:** ASMP-22
- **Payment-processing capability** — Required to run the paid household subscription (monthly or yearly); without it, the product could offer only the free tier and would have no path to the revenue the founder needs within about three months.
- **ID:** ASMP-23
- **Recipe/food-content data capability** — Required to seed the starter recipe library at launch; without it, a brand-new household would have to rely entirely on manually imported recipes before any plan could be generated.
- **ID:** ASMP-24
- **Real-time data-synchronization capability** — Required for the shared grocery list to update live across household members' devices and to reconcile changes made while offline; without it, the list would revert to the unreliable, manually-reconciled experience the brief describes households currently suffering through.
