---
document_type: scope-boundaries
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed
---

# Scope Boundaries

## In-Scope Summary

Plateful enables one household to set up its members, allergies, diets, budget, and schedule once, then plan a week of dinners that respects every hard dietary rule — by hand on the free tier, or as a weekly AI-generated 7-day plan on the paid tier that also uses up what's already on hand — and produces one shared, aisle-grouped grocery list. Core features cover setup, allergy safety, AI plan generation, manual weekly planning, swapping, pantry awareness, the shared list, the plan-ready notification, the starter recipe library, and household membership; Important features add recipe import, leftover rollover, ratings and learning, reminders, billing, onboarding, household-to-household invitations, the weekly waste and spend check-in, localization, older-kid voting, and account/data controls; Nice-to-Have features add plan history, two later-phase integrations, and limited operator support access. The product serves one household per account, with everyone in that household seeing the same plan and list. [MODIFIED: summary updated for the three features added in synthesis — Manual Weekly Planning (FEAT-23), Invite Another Household (FEAT-24), Weekly Waste & Spend Check-In (FEAT-25)]

## Explicit Exclusions

### User Scope Exclusions

- **ID:** SC-01
- **Operator as a full product role** — BRIEF.md's Target Users & Roles section states the founder's operator role is explicitly "not a product role" and needs "only read-only support access... Nothing more" — it is never modeled as a persona with product-facing entitlements beyond that.
- **ID:** SC-02
- **Independent kid login as the v1 default** — The persona set's Kid Profile confirms the founder's leaning toward "parent-managed profiles with no login for young kids" as the default; any kid login is a distinct, Later-phase possibility (FEAT-17), not a v1 baseline.
- **ID:** SC-03
- **Multiple households per account** — BRIEF.md's Target Users & Roles section states plainly "there is one household per account in v1"; managing more than one household from a single account is out of scope.
- **ID:** SC-04
- **Other adult members changing household setup, approving the plan, or swapping without the organiser** — BRIEF.md's Target Users & Roles section gives setup, dietary rules, budget, schedule, and plan approval to the organiser, while other adult members "see the plan and suggest swaps"; the persona set's Access Matrix reflects this, and the organiser role can be handed over (FEAT-09) rather than shared. [AUDIT-EXCLUDED: 1 -- journey walk: the draft let other adults swap directly, which the brief's role split does not support]

### Feature Scope Exclusions

- **ID:** SC-05
- **Native mobile applications** — BRIEF.md's Constraints section states the platform is "a responsive web app for v1, with no native apps and no app stores," a deliberate solo-founder shipping-speed choice.
- **ID:** SC-06
- **Medical or diet advice, including calorie/macro guidance as a recommendation engine** — BRIEF.md states directly "there is no medical or diet advice" and flags nutrition information as an open question the founder has not resolved toward inclusion; the product presents cost, time, and safety information, not dietary prescriptions. [RESEARCH-INFORMED: competitors' nutrition tracking and automatic "Health Score" labels drew recurring criticism as diet-culture language (Samsung Food user reports, MEDIUM confidence), which supports keeping health scoring out]
- **ID:** SC-07
- **Meal-kit box sales or grocery delivery of physical ingredients** — BRIEF.md's Problem Statement explicitly positions Plateful against "meal-kit companies want to sell boxes"; the product plans and lists groceries, it does not sell or ship ingredients itself.
- **ID:** SC-08
- **Advertising as a revenue feature** — BRIEF.md's Business Context states "there are no ads, ever," named as a deliberate selling point given parents' sensitivity to it. [RESEARCH-INFORMED: intrusive free-tier ads are a recurring complaint about a family organizer (Cozi independent review, MEDIUM confidence)]
- **ID:** SC-09
- **Cross-household social or community features (recipe sharing feeds, public profiles)** — BRIEF.md's Ecosystem & Integrations confirms "otherwise the product stands alone"; growth comes from direct household-to-household invite links (FEAT-24), not a social layer. [MODIFIED: growth reference moved from Household Invitations & Membership (FEAT-09) to Invite Another Household (FEAT-24), which now owns household-to-household growth]
- **ID:** SC-10
- **Rewards, credits, or discounts for inviting other households** — BRIEF.md's Business Context expects households to invite households but names no incentive; Invite Another Household (FEAT-24) records referrals without paying for them, keeping money flows limited to the household subscription. [AUDIT-EXCLUDED: 1 -- value-flow walk: a referral reward would open a new money flow the brief never asked for]
- **ID:** SC-11
- **Detailed pantry inventory tracking (quantities, expiry dates, barcode stock-taking)** — BRIEF.md's Open Questions asks whether to track a detailed inventory or "just 'use up what I tell you I have'," and the product follows the lighter model in Pantry-Aware Suggestions (FEAT-05). [AUDIT-EXCLUDED: 2 -- competitive cross-reference: the one competitor with pantry management is criticised for missing real kitchen behavior such as leftovers (Samsung Food, MEDIUM confidence), a gap Plateful answers with Leftover Rollover to Lunches (FEAT-11) rather than a heavier inventory]
- **ID:** SC-12
- **Importing plans, lists, or recipe collections in bulk from other meal-planning or list apps** — the brief names only per-link recipe import (BRIEF.md, Ecosystem & Integrations); households bring recipes in one link at a time (FEAT-10) and add list items by hand. [AUDIT-EXCLUDED: 4 -- Data Import: no brief goal or HIGH-confidence evidence supports bulk import, and it would add a data path the solo founder must maintain]
- **ID:** SC-13
- **Languages other than English** — BRIEF.md's Scale & Non-Functional Expectations targets the US and UK first, requiring configurable units, currency, and aisle names (FEAT-16) but not translation. [AUDIT-EXCLUDED: 4 -- Internationalization: locale formats are in scope; translated content is not needed for the two launch markets]
- **ID:** SC-14
- **In-product messaging or chat between household members** — BRIEF.md's Problem Statement names the group chat as the problem to replace for the grocery list, not as a feature to rebuild; the shared list, swap suggestions, and notifications carry the coordination the household needs. [AUDIT-EXCLUDED: 4 -- Roles and Sharing: coordination is covered by features, not a messaging layer]

### Scale Expectations

- **ID:** SC-15
- **Household volume** — Expected: several thousand households in the first year, 2-6 people each (BRIEF.md, Scale & Non-Functional Expectations); the product is designed to stay responsive at this order of magnitude from MVP onward, not scaled up gradually from a much smaller baseline.
- **ID:** SC-16
- **AI cost per household** — Expected: roughly one weekly plan generation plus a few swaps per household per week, keeping AI cost small enough for the founder's sub-$100/month pre-revenue infrastructure budget (BRIEF.md, Scale & Non-Functional Expectations, Constraints: Budget); this bound holds from MVP and does not loosen as household count grows. Free-tier households, the default for every new household, generate no AI cost (BRIEF.md, Business Context). [MODIFIED: free-tier zero-AI-cost made explicit alongside Manual Weekly Planning (FEAT-23)]
- **ID:** SC-17
- **Grocery list responsiveness at scale** — Expected: the shared grocery list stays instant-feeling and keeps working through supermarket connectivity gaps regardless of household size, from MVP onward (BRIEF.md, Scale & Non-Functional Expectations).
- **ID:** SC-18
- **History depth** — Expected: every past week's plan, list, ratings, and check-in answers are kept for the life of the household account and remain available after a downgrade to the free tier; they are browsable from v1 (Weekly Plan History, FEAT-19) and exportable from MVP (FEAT-18). [RESEARCH-INFORMED: retroactively paywalling users' own history produced a documented trust backlash (Cozi, Trustpilot average 2.1/5, HIGH confidence)]
- **ID:** SC-19
- **Enterprise, institutional, or catering-scale meal planning** — Genuinely out of vision at any scale: the product is scoped to individual households (BRIEF.md, Target Users & Roles, Business Context); planning for schools, care facilities, or catering businesses would require a fundamentally different rule set and is not a future phase of this product.

## Deferral Notes

- **Recipe Import from Web Link (FEAT-10)** — Target phase: v1. Deferred rather than rejected: the starter recipe library (FEAT-08) gives new households a working candidate pool from day one; import extends personalization once the core loop is proven and the brief's open legality question is resolved. [RESEARCH-INFORMED: import is a common competitor feature (AnyList, Samsung Food) whose reliability problems are well reported, so it should ship with review-before-save and manual fallback rather than rushed into MVP]
- **Weekly Plan History (FEAT-19)** — Target phase: v1. Useful once several weeks of plans have accumulated; not needed for the core weekly loop to function. The underlying history is kept from MVP (SC-18), so nothing is lost before browsing ships.
- **Operator Read-Only Support Access (FEAT-22)** — Target phase: v1. The product can be supported manually at very small scale at launch; a dedicated read-only view becomes worth building once household volume makes ad hoc support impractical. Households can already raise safety concerns (FEAT-02) and contact support (FEAT-18) from MVP.
- **Older-Kid Dinner Voting (FEAT-17)** — Target phase: Later. Deferred until the brief's open question on how older kids are represented (limited login vs. some other mechanism) is resolved; voting depends on that representation existing first.
- **Online Grocery Ordering Handoff (FEAT-20)** — Target phase: Later. Named directly in BRIEF.md as "desirable later, not v1"; worth revisiting once the core plan-and-list loop has proven itself and a regional ordering capability is identified.
- **Family Calendar Sync (FEAT-21)** — Target phase: Later. Named directly in BRIEF.md as "a nice-to-have... not v1"; an additive integration that does not change the core product if it never ships.
- **Breakfast and full lunch planning** — Target phase: Later. BRIEF.md's Open Questions asks "breakfast and lunch too, or dinners only for v1?"; the product plans dinners only, with lunches covered solely by Leftover Rollover to Lunches (FEAT-11). It becomes worth a feature entry once households show — through check-in answers or support requests — that dinners-only planning leaves meaningful waste or planning effort at other meals. [AUDIT-ADDED: 4 -- the brief's meal-scope open question had no recorded decision in the draft]
