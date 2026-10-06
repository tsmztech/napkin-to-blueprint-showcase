# Operating Rules

These rules are binding for any agent or human building from this workspace.

## 1. The recommended architecture is binding

`docs/blueprint/architecture/technical-architecture.md` records a RECOMMENDED choice for
every decision area, plus documented alternatives with `Choose instead when` conditions.
Build the recommendation. The alternatives are informational — documented for the humans
who own this project to weigh; they are not the agent's to choose. If a human explicitly
directs a substitution, record it in `PROGRESS.md` and apply it consistently.

## 2. DO-NOT-BUILD — explicit scope exclusions

The exclusions below are carried verbatim from the blueprint
(`docs/blueprint/features/scope-boundaries.md`). Never implement any of them — even when a
spec seems adjacent or the capability seems easy to add. Deferral notes live in the source
document.

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


## 3. Assumptions & constraints awareness

`docs/blueprint/features/assumptions-constraints.md` carries the ASMP register — the
assumptions this blueprint is built on — plus product constraints, non-functional
expectations, and dependencies. Read it before architectural or data-model work; when an
implementation choice would contradict an ASMP entry, stop and surface it to a human
instead of silently deciding.

## 4. The blueprint is read-only

Everything under `docs/blueprint/` is input, never workspace: never edit, reformat,
regenerate, or delete a blueprint file. A blueprint defect is reported to a human (the
package is regenerated upstream) — it is not patched here. Specs are the contract: build
what they say, and when a spec and this file's rules seem to conflict, these rules win and
the conflict is reported.

## 5. Design posture

This package ships design-agnostic: no design system is part of the blueprint, and the
builder owns visual design. Honor any stated design preferences recorded in the brief's
Constraints section (`docs/blueprint/BRIEF.md`).
