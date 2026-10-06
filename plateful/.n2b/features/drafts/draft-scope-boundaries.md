---
document_type: scope-boundaries
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# Scope Boundaries

## In-Scope Summary

Plateful enables one household to set up its members, allergies, diets, budget, and schedule once, then receive a weekly AI-generated 7-day dinner plan that respects every hard dietary rule, uses up what's already on hand, and produces one shared, aisle-grouped grocery list. Core features cover setup, allergy safety, plan generation, swapping, pantry awareness, and the shared list; Important features add recipe import, leftover rollover, learning from ratings, reminders, billing, onboarding, localization, older-kid voting, and account/data controls; Nice-to-Have features add plan history and two later-phase integrations plus limited operator support access. The product serves one household per account, with everyone in that household seeing the same plan and list.

## Explicit Exclusions

### User Scope Exclusions

- **ID:** SC-01
- **Operator as a full product role** — BRIEF.md's Target Users & Roles section states the founder's operator role is explicitly "not a product role" and needs "only read-only support access... Nothing more" — it is never modeled as a persona with product-facing entitlements beyond that.
- **ID:** SC-02
- **Independent kid login as the v1 default** — The persona set's Kid Profile confirms the founder's leaning toward "parent-managed profiles with no login for young kids" as the default; any kid login is a distinct, Later-phase possibility (FEAT-17), not a v1 baseline.
- **ID:** SC-03
- **Multiple households per account** — BRIEF.md's Target Users & Roles section states plainly "there is one household per account in v1"; managing more than one household from a single account is out of scope.

### Feature Scope Exclusions

- **ID:** SC-04
- **Native mobile applications** — BRIEF.md's Constraints section states the platform is "a responsive web app for v1, with no native apps and no app stores," a deliberate solo-founder shipping-speed choice.
- **ID:** SC-05
- **Medical or diet advice, including calorie/macro guidance as a recommendation engine** — BRIEF.md states directly "there is no medical or diet advice" and flags nutrition information as an open question the founder has not resolved toward inclusion; the product presents cost, time, and safety information, not dietary prescriptions.
- **ID:** SC-06
- **Meal-kit box sales or grocery delivery of physical ingredients** — BRIEF.md's Problem Statement explicitly positions Plateful against "meal-kit companies want to sell boxes"; the product plans and lists groceries, it does not sell or ship ingredients itself.
- **ID:** SC-07
- **Advertising as a revenue feature** — BRIEF.md's Business Context states "there are no ads, ever," named as a deliberate selling point given parents' sensitivity to it.
- **ID:** SC-08
- **Cross-household social or community features (recipe sharing feeds, public profiles)** — BRIEF.md's Ecosystem & Integrations confirms "otherwise the product stands alone"; growth comes from direct household-to-household invitations (FEAT-09), not a social layer.

### Scale Expectations

- **ID:** SC-09
- **Household volume** — Expected: several thousand households in the first year, 2-6 people each (BRIEF.md, Scale & Non-Functional Expectations); the product is designed to stay responsive at this order of magnitude from MVP onward, not scaled up gradually from a much smaller baseline.
- **ID:** SC-10
- **AI cost per household** — Expected: roughly one weekly plan generation plus a few swaps per household per week, keeping AI cost small enough for the founder's sub-$100/month pre-revenue infrastructure budget (BRIEF.md, Scale & Non-Functional Expectations, Constraints: Budget); this bound holds from MVP and does not loosen as household count grows.
- **ID:** SC-11
- **Grocery list responsiveness at scale** — Expected: the shared grocery list stays instant-feeling and keeps working through supermarket connectivity gaps regardless of household size, from MVP onward (BRIEF.md, Scale & Non-Functional Expectations).
- **ID:** SC-12
- **Enterprise, institutional, or catering-scale meal planning** — Genuinely out of vision at any scale: the product is scoped to individual households (BRIEF.md, Target Users & Roles, Business Context); planning for schools, care facilities, or catering businesses would require a fundamentally different rule set and is not a future phase of this product.

## Deferral Notes

- **Recipe Import from Web Link (FEAT-10)** — Target phase: v1. Deferred rather than rejected: the starter recipe library (FEAT-08) gives new households a working candidate pool from day one; import extends personalization once the core loop is proven and the brief's open legality question is resolved.
- **Meal Rating & Preference Learning (FEAT-12)** — Target phase: v1. Deferred because meaningful learning requires a history of ratings that does not exist in a household's first weeks; MVP plan generation is designed to work well without it.
- **Weekly Plan History (FEAT-19)** — Target phase: v1. Useful once several weeks of plans have accumulated; not needed for the core weekly loop to function.
- **Operator Read-Only Support Access (FEAT-22)** — Target phase: v1. The product can be supported manually at very small scale at launch; a dedicated read-only view becomes worth building once household volume makes ad hoc support impractical.
- **Older-Kid Dinner Voting (FEAT-17)** — Target phase: Later. Deferred until the brief's open question on how older kids are represented (limited login vs. some other mechanism) is resolved; voting depends on that representation existing first.
- **Online Grocery Ordering Handoff (FEAT-20)** — Target phase: Later. Named directly in BRIEF.md as "desirable later, not v1"; worth revisiting once the core plan-and-list loop has proven itself and a regional ordering capability is identified.
- **Family Calendar Sync (FEAT-21)** — Target phase: Later. Named directly in BRIEF.md as "a nice-to-have... not v1"; an additive integration that does not change the core product if it never ships.
