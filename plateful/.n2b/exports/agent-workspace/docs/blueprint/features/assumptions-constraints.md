---
document_type: assumptions-constraints
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed
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
- **ID:** ASMP-04
- **We assume enough household members will either enable device notifications for the web app or accept the email fallback that the Sunday plan-ready message reliably reaches them.** Invalidated if: fewer than half of organisers receive the plan-ready message by either route in their first month. [AUDIT-ADDED: 4 -- External-Service Integrations: device notifications from a web app are not available on every phone, so the weekly rhythm rests on this assumption]

### User Behavior

- **ID:** ASMP-05
- **We assume the organiser reviews and actively engages with the AI-generated plan each week rather than ignoring it.** Invalidated if: usage data shows a large share of generated plans are never opened.
- **ID:** ASMP-06
- **We assume other adult household members actively use the shared grocery list themselves, rather than the organiser doing all the shopping and list management alone.** Invalidated if: list activity (ticks, manual adds) is concentrated almost entirely on the organiser's account across most households.
- **ID:** ASMP-07
- **We assume households are willing and able to enter accurate allergy and dietary-rule information during setup.** Invalidated if: support requests or safety reports reveal a pattern of incomplete or incorrect allergy data being entered at setup time.
- **ID:** ASMP-08
- **We assume the organiser answers other adults' swap suggestions quickly enough that routing swaps through the organiser does not become a bottleneck.** Invalidated if: a large share of swap suggestions lapse unanswered before their night. [AUDIT-ADDED: 1 -- the suggest-and-approve role split taken from BRIEF.md, Target Users & Roles depends on it]
- **ID:** ASMP-09
- **We assume most households will answer a one-question weekly waste and spend check-in often enough to show a trend.** Invalidated if: fewer than a third of active households answer at least two check-ins in their first month, leaving the food-waste success criterion unmeasurable. [AUDIT-ADDED: 3 -- Weekly Waste & Spend Check-In relies on voluntary answers]

### Product Context

- **ID:** ASMP-10
- **We assume Plateful can deliver its core value as a standalone product, without needing deep integration into an existing grocery-ordering or meal-kit ecosystem at launch.** Invalidated if: early adoption stalls specifically because households want online ordering integration (FEAT-20) before they will commit to the product.
- **ID:** ASMP-11
- **We assume growth will come primarily from household-to-household invitations rather than paid acquisition, at least in the first year.** Invalidated if: invitation-driven signups remain a small minority of new paying households after several months of operation.
- **ID:** ASMP-12
- **We assume families will trust a new, standalone subscription meal planner even though several planning apps have recently shut down.** Invalidated if: prospective households repeatedly cite worries that the product will disappear as a reason not to subscribe. [RESEARCH-INFORMED: Yummly, PlateJoy, and Mealime closed or were folded into other products within about two years, and Mealime's own users are being displaced in October 2026 (market research, MEDIUM confidence)]

## Product Constraints

- **ID:** ASMP-13
- **Allergies and religious rules are hard filters enforced by the app itself, never left to AI judgment alone.** BRIEF.md's Constraints section states the AI "is never the last line of defence" — this is a deliberate safety architecture choice, not a feature that could be relaxed for convenience. The check covers every path onto the plan — AI suggestions, swaps, manual picks, and re-used past weeks — fails closed when ingredient data is incomplete, and every meal carries the "always check labels" disclaimer. [MODIFIED: coverage of every path onto the plan and fail-closed behavior made explicit, matching the final Dietary Rules & Allergy Safety Engine]
- **ID:** ASMP-14
- **No advertising and no sale or third-party use of family data, ever.** BRIEF.md's Business Context names this directly as a deliberate trust-building positioning constraint, not merely a launch-time omission that might change with monetization pressure.
- **ID:** ASMP-15
- **Responsive web app only in v1, with no native apps and no app stores.** BRIEF.md's Constraints section states this is a deliberate solo-founder, ship-fast choice.
- **ID:** ASMP-16
- **One household per account in v1.** BRIEF.md's Target Users & Roles section states this directly as a deliberate scope constraint that keeps the account and access model simple at launch.
- **ID:** ASMP-17
- **No medical or diet advice is given by the product.** BRIEF.md's Constraints section states this explicitly, positioning Plateful as a planning tool rather than a health or medical product.
- **ID:** ASMP-18
- **The free tier includes no AI; manual planning and the shared grocery list are free, and the AI plan, pantry-aware suggestions, and learning from ratings are paid.** BRIEF.md's Business Context and Scale & Non-Functional Expectations set this split, which also keeps AI cost tied to paying households. [AUDIT-ADDED: 4 -- Monetization and Billing Touchpoints: the tier boundary shapes several features and was not recorded as a constraint]
- **ID:** ASMP-19
- **Nothing a household can already use for free is ever moved behind the paywall, and a downgrade never takes away a household's own history.** A deliberate trust choice for a product whose buyers are privacy- and fairness-sensitive parents (BRIEF.md, Business Context). [RESEARCH-INFORMED: a family organizer's 2024 move of users' own calendar history behind a paywall produced a Trustpilot average of 2.1/5 and "bait and switch" language (Cozi, HIGH confidence)]
- **ID:** ASMP-20
- **Dinners only, with leftovers rolling into lunches.** BRIEF.md's Open Questions leaves breakfast and lunch planning open; the product plans dinners and treats lunch only through Leftover Rollover to Lunches (FEAT-11), a deliberate scope choice for the three-month timeline (BRIEF.md, Constraints: Team / timeline). [AUDIT-ADDED: 4 -- the brief's meal-scope open question had no recorded decision]
- **ID:** ASMP-21
- **Other adult members suggest swaps and the organiser approves the plan.** BRIEF.md's Target Users & Roles gives setup, budget, dietary rules, and plan approval to the organiser and has other adults "suggest swaps"; the product keeps that split rather than giving every adult equal control. [AUDIT-ADDED: 1 -- role split recorded as a constraint because it shapes One-Tap Meal Swap and Manual Weekly Planning]

## Non-Functional Expectations

- **ID:** ASMP-22
- **Responsiveness: the shared grocery list feels instant when items are ticked or added, and a meal swap completes and reflects in the list within seconds.** Basis: BRIEF.md's Scale & Non-Functional Expectations section states the list "must feel instant," and the Vision states swaps update the list "instantly."
- **ID:** ASMP-23
- **Responsiveness for heavier moments: a weekly plan generates within well under a minute with an explained wait, and recipe search results appear within about a second.** Basis: the States fields of AI Weekly Dinner Plan Generation (FEAT-03) and Recipe Library (FEAT-08), and the brief's under-10-minutes-a-week planning goal (BRIEF.md, Success Criteria). [AUDIT-ADDED: 4 -- Non-Functional Expectations: the draft covered list and swap responsiveness only]
- **ID:** ASMP-24
- **Data volume and growth: several thousand households in the first year, 2-6 members each, with the product staying equally responsive as that base grows; every household's history is kept for the life of the account.** Basis: BRIEF.md's Scale & Non-Functional Expectations, Order of magnitude; history depth per scope-boundaries.md SC-18. [MODIFIED: history depth added to match the scale expectations in scope-boundaries.md]
- **ID:** ASMP-25
- **Resilience: the shared grocery list keeps working, with changes queued and synced later, when used in a supermarket with poor or no signal; an already-generated plan stays viewable offline.** Basis: BRIEF.md's Scale & Non-Functional Expectations, Performance/availability.
- **ID:** ASMP-26
- **Privacy posture: children's data is minimal, parent-controlled, and used only for the household's own plan; no data is ever sold or used for advertising to any household member. A kid profile holds only a first name or nickname, an age band, and dietary rules; the organiser can see every time support viewed the household.** Basis: BRIEF.md's Privacy section under Scale & Non-Functional Expectations and Constraints. [MODIFIED: kid-profile data limits and support-access visibility added, matching Household Setup & Member Profiles and Operator Read-Only Support Access]
- **ID:** ASMP-27
- **Compliance: because the product processes children's data in the US and UK, it is treated as subject to children's-privacy-class protections (minimal collection, verifiable parental consent when a kid profile is created, parental control, no behavioral advertising to minors) alongside general personal-data protection for all household members, including the right to a copy of their data and to deletion; no medical-data regime applies, since the product gives no medical or diet advice.** Basis: domain reasoning from decomposition-checklists.md's Compliance section, grounded in BRIEF.md's stated children's-privacy sensitivity and its explicit "no medical or diet advice" positioning. [MODIFIED: parental consent at kid-profile creation and data-subject rights (export, deletion) tied to the features that deliver them]
- **ID:** ASMP-28
- **Localization: measurement units, currency, and supermarket aisle names are configurable per household to support both US and UK households from launch; the product is in English only.** Basis: BRIEF.md's Scale & Non-Functional Expectations, Geography; English-only per scope-boundaries.md SC-13. [MODIFIED: English-only recorded to match the scope exclusion]
- **ID:** ASMP-29
- **Accessibility and one-handed use: every primary action is reachable with one thumb on a phone, tap targets are large, text stays readable with the phone's larger-text settings, and safety badges and ineligibility reasons are conveyed in words, not by color alone.** Basis: BRIEF.md's Constraints, Design preference ("big tap targets for one-handed use in a shop"). [AUDIT-ADDED: 4 -- Accessibility Baseline: a product-level baseline was missing from the draft]

## Dependencies

- **ID:** ASMP-30
- **AI text/plan-generation capability** — Required to generate the weekly dinner plan and swap alternatives on the paid tier; without it, planning would revert to a fully manual, unassisted experience, and the product's paid promise would not exist.
- **ID:** ASMP-31
- **Device-notification delivery capability** — Required for the weekly "plan ready" notification, the daily "tonight's dinner" nudge, and swap-suggestion alerts; without it, these degrade to email (plan ready) or in-app-only discovery, weakening the product's proactive rhythm.
- **ID:** ASMP-32
- **Transactional email capability** — Required for account sign-up and sign-in recovery, the plan-ready email fallback, billing and grace-period notices, data-export and deletion confirmations, safety-concern reports reaching the operator, and support acknowledgements; without it, several account, billing, and safety messages would have no reliable route. [AUDIT-ADDED: 4 -- External-Service Integrations: many features send email, but the draft listed no email dependency]
- **ID:** ASMP-33
- **Payment-processing capability** — Required to run the paid household subscription (monthly or yearly); without it, the product could offer only the free tier and would have no path to the revenue the founder needs within about three months.
- **ID:** ASMP-34
- **Recipe/food-content data capability** — Required to seed the starter recipe library at launch with complete ingredient data the allergy check can verify; without it, a brand-new household would have to rely entirely on manually imported recipes before any plan could be generated.
- **ID:** ASMP-35
- **Real-time data-synchronization capability** — Required for the shared grocery list and plan to update live across household members' devices and to reconcile changes made while offline; without it, the list would revert to the unreliable, manually-reconciled experience the brief describes households currently suffering through.
- **ID:** ASMP-36
- **Web-page recipe extraction capability (v1)** — Required for Recipe Import from Web Link (FEAT-10) to read a recipe's ingredients and steps from a pasted link; without it, households could still add recipes by manual entry. Whether importing from other sites is legally acceptable, and in what form, remains a brief open question to settle before v1 (BRIEF.md, Open Questions). [AUDIT-ADDED: 4 -- External-Service Integrations: capability needed by FEAT-10 was not listed]
- **ID:** ASMP-37
- **Online grocery-ordering and family-calendar capabilities (Later)** — Required only for Online Grocery Ordering Handoff (FEAT-20) and Family Calendar Sync (FEAT-21); the core product works fully without them. [AUDIT-ADDED: 4 -- External-Service Integrations: Later-phase dependencies recorded so Stage 4 can plan for them]
