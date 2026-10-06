---
document_type: user-persona
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed
---

# User Personas

## Persona Set Summary

This product serves three distinct household roles plus one non-product support role. The primary persona is the household organiser, who sets up the household and approves the weekly plan (BRIEF.md, Target Users & Roles). Two secondary personas — other adult household members and kid profiles — reflect the brief's explicit description of a shared household plan and grocery list used by everyone in the household. A fourth, lightweight entry captures the founder's read-only operator support role, which the brief itself names but explicitly marks "not a product role." The kid role is modeled in two forms — a no-login young-kid profile (the MVP default) and a Later-phase limited login for older kids — because the brief gives them genuinely different entitlements. [MODIFIED: kid role split into its two brief-stated forms so the Access Matrix no longer grants list and rating access to a profile that has no login (BRIEF.md, Target Users & Roles, Open Questions)]

## Primary Persona

### Persona Name

Maya

### Description

Maya is a working parent with two kids, one of whom has a food allergy, and a partner on a different diet. She is the household organiser: the person who creates the household in Plateful, invites the rest of the family, and sets up everyone's dietary rules, the weekly budget, and the household's schedule. She currently plans meals with recipe screenshots, a notes app, and a group chat, and still ends up with mid-week takeaway and food thrown out by the weekend (BRIEF.md, Problem Statement).

### Goals

- Get a realistic 7-day dinner plan every week without spending more than about 10 minutes on it (BRIEF.md, Success Criteria, Vision)
- Keep every suggested meal genuinely safe for her child's allergy, with no exceptions (BRIEF.md, Constraints: Safety)
- Use up food that is already in the fridge instead of throwing it away (BRIEF.md, Vision)
- Keep the household's grocery list in one shared place instead of a group chat three people edit (BRIEF.md, Problem Statement)
- Approve or adjust the week's plan quickly when it does not quite fit, including acting on the swaps her partner suggests (BRIEF.md, The Experience, Target Users & Roles) [MODIFIED: suggestions from other adults added, per the brief's role split — other adults suggest swaps, the organiser approves the plan]

### Pain Points

- Sunday-night planning is a recurring chore that has to juggle an allergy, a partner's diet, a picky eater, and 30-minute weeknights all at once (BRIEF.md, Problem Statement)
- Existing "AI meal plan" tools ignore real constraints — one suggested a walnut salad to a family with a nut allergy (BRIEF.md, Problem Statement) [RESEARCH-INFORMED: added market evidence that allergy handling across existing planners is shallow and exclusion-based, entering several allergies "left her with very few options," and no product enforces an app-level allergy check (App Store and independent reviews across 4 products, HIGH confidence)]
- Recipe sites are ad-stuffed and meal-kit companies are trying to sell boxes, not solve the actual planning problem (BRIEF.md, Problem Statement)
- The current grocery list lives in a group chat that three people edit, which is unreliable and easy to lose track of (BRIEF.md, Problem Statement)
- Despite all the planning effort, the household still ends up with mid-week takeaway and food in the bin by Saturday (BRIEF.md, Problem Statement)

### Behavioral Context

Maya's main touchpoint is a Sunday-evening notification that the next week's plan is ready, which she opens on her phone from the sofa. She may use a bigger screen (a laptop) for the initial household setup, since that involves more detailed input (members, allergies, budget, schedule), but day-to-day interaction — reviewing the plan, swapping a meal, approving changes — happens on her phone in short sessions (BRIEF.md, Target Users & Roles, Scale & Non-Functional Expectations). Before upgrading, she plans the same Sunday slot by hand on the free tier. [MODIFIED: free-tier manual planning added to reflect that every household starts on the free tier (BRIEF.md, Business Context)]

### What This User Does NOT Need

- A detailed nutrition or calorie-tracking tool — the brief explicitly states no medical or diet advice is intended (BRIEF.md, Open Questions, Constraints) [RESEARCH-INFORMED: a competitor's automatic "Health Score" labeling drew recurring criticism as diet-culture language (Samsung Food user reports, MEDIUM confidence)]
- Native mobile apps — v1 is a responsive web app only (BRIEF.md, Constraints: Platform)
- Meal-kit box ordering or ads inside the product — the brief positions Plateful against both (BRIEF.md, Problem Statement, Business Context)
- Multi-household management — there is one household per account in v1 (BRIEF.md, Target Users & Roles)

## Secondary Personas

### Other Adult Member

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section names "other adult members (partner, roommate, a grandparent who cooks on Tuesdays)" as people who see the plan, suggest swaps, tick off the grocery list, and rate meals — a distinct entitlement set from the organiser's setup-and-approve role. [RESEARCH-INFORMED: household-level access distinct from a single shared list is an established pattern — AnyList prices a household tier separately and Samsung Food offers household invites, while Mealime's lack of household sharing is flagged by reviewers as a family-use limitation (competitor profiles, HIGH confidence)]

**Name:** Sam

**Description:** Sam is Maya's partner (or, in another household, a roommate or a grandparent who cooks regularly). Sam follows a different diet than the rest of the household and shares in cooking and shopping duties, but did not set up the household and does not manage its dietary rules or budget.

**Goals:** See the week's plan without having to ask; suggest a swap when a night doesn't work for him and see whether Maya accepted it; top up the shared grocery list from wherever they are; tick items off while actually in the supermarket, even with a weak signal; rate meals honestly after dinner so future plans improve. [MODIFIED: "suggest a swap" goal added, per BRIEF.md, Target Users & Roles]

**Pain Points:** The current grocery list is a group chat three people edit, which is easy to lose track of (BRIEF.md, Problem Statement); Sam has no reliable way today to see what's already been decided for the week without checking with Maya. [RESEARCH-INFORMED: where competitors combine plans with household sharing, members report seeing the week's plan but not the matching list (Samsung Food, MEDIUM confidence) — the reliability Sam needs is not a given]

**Behavioral Context:** Sam opens the app with the shared list already loaded while standing in the supermarket, one hand on a trolley; also uses it at the evening "tonight's dinner" nudge and right after dinner to rate the meal, passing his phone to the kids so they can rate too (BRIEF.md, The Experience).

**What This User Does NOT Need:** Household setup screens, budget configuration, dietary-rule administration, plan approval, or billing — those belong to the organiser (BRIEF.md, Target Users & Roles).

### Kid Profile

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section states "kids' allergies and dislikes drive the plan, and older kids want to vote on dinners," and that young kids should not need their own account, with parent-managed profiles as the founder's leaning; this is a genuinely distinct entitlement set (little or no login, no setup access) from either adult role. BRIEF.md's Open Questions leaves a limited login for older kids as a later possibility, so the Access Matrix carries two kid rows: a no-login young-kid profile (MVP) and an older-kid limited login (Later, tied to Older-Kid Dinner Voting). [MODIFIED: two kid rows made explicit in the provenance to match the Access Matrix]

**Name:** Jordan

**Description:** Jordan is one of Maya's kids. As a younger child, Jordan has no login at all — Maya manages Jordan's allergy and dislike profile on their behalf. As an older kid (a distinct sub-case the brief leaves open for a possible future limited login), Jordan might vote on dinners, rate meals, and add to the list directly.

**Goals:** Have their allergy and dislikes genuinely respected in every suggested meal; give a thumbs up or down after dinner (on a parent's phone while young); as an older kid, have a say in which dinner gets picked and add small things to the shared list ("more yoghurt"). [MODIFIED: the draft had young kids adding to the list "without needing an account", which a no-login profile cannot do; list additions now sit with the Later-phase older-kid login, matching the brief's "teenager" in The Experience]

**Pain Points:** Existing "AI meal plan" tools do not reliably respect a stated allergy, which is exactly the risk this persona embodies most directly (BRIEF.md, Problem Statement).

**Behavioral Context:** For young kids, all interaction is indirect — Maya enters and maintains Jordan's allergy/dislike data, and an adult records Jordan's post-dinner thumbs up or down on their own phone. For an older kid with a limited login (a Later-phase possibility), interaction is occasional: voting on the week's dinners, adding an item to the list from the couch, and a post-dinner thumbs up/down (BRIEF.md, The Experience, Target Users & Roles).

**What This User Does NOT Need:** A full account, household setup access, budget or dietary-rule administration, notifications, or any login at all as a young child — the founder's stated default is parent-managed profiles with no login for young kids (BRIEF.md, Target Users & Roles, Open Questions). Kids' data stays minimal: a first name or nickname, an age band, and dietary rules (BRIEF.md, Constraints: Privacy / children).

### Operator (Support)

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section names "Operator support (not a product role)" explicitly, stating "the founder, as operator, needs only read-only support access to help a household. Nothing more." This is included here, lightly, purely because the brief names it directly and grounded-roles requires every access level in the product to trace to a cited source — it is not part of the product's user-facing role model.

**Name:** Riley

**Description:** Riley is the founder acting in a support capacity, occasionally helping a household troubleshoot a problem with their plan or list, or reviewing a safety concern a household has reported about a meal.

**Goals:** See enough of a household's setup and plan to diagnose a reported problem, without being able to change anything or see more than necessary; resolve safety-concern reports quickly.

**Pain Points:** N/A — this is a support capability, not a persona with product pain points of their own.

**Behavioral Context:** Rare, on-demand use triggered by a household's support request or safety-concern report; never a routine or scheduled interaction, and each visit is visible to the household's organiser. [AUDIT-ADDED: 4 -- Audit Logging and Security and Privacy Posture: support access opens only against a household-raised request and leaves a record the organiser can see]

**What This User Does NOT Need:** Any ability to edit household data, view billing details beyond plan tier, or see children's dietary data beyond what is strictly needed to diagnose an allergy-safety report — the brief's privacy constraint on children's data applies here too (BRIEF.md, Constraints: Privacy).

## Access Matrix

Access levels: **Full** — can see and do everything in the group; **View** — can see but not change; **Own-only** — can act only on their own contributions (for Meal Swap and Manual Planning this means raising their own suggestions, which the organiser accepts or declines); **None** — no access, and the group is not shown. Riley's access exists only from v1 (Operator Read-Only Support Access). [MODIFIED: matrix extended during the Access Matrix audit — columns added for Manual Planning, Household Invitations, Household Referrals, Account & Data, Safety Reports, Waste Check-In, Dinner Voting and Support View to cover capability groups introduced or made explicit in synthesis; the kid row split into young-kid and older-kid rows; Sam's Meal Swap changed from Full to Own-only per BRIEF.md's "suggest swaps"]

| Role / Persona | Household Setup | Weekly Plan | Meal Swap | Manual Planning | Pantry Input | Grocery List | Recipe Library | Ratings | Notification Prefs | Household Invitations | Household Referrals | Billing | Account & Data | Kid Profile Data | Safety Reports | Waste Check-In | Dinner Voting | Support View |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Maya (Organiser) | Full | Full | Full | Full | Full | Full | Full | Full | Full | Full | Full | Full | Full | Full | Full | Full | Full | View |
| Sam (Other Adult Member) | View | View | Own-only | Own-only | Full | Full | Full | Own-only | Own-only | None | Full | None | Own-only | View | Own-only | Full | View | None |
| Jordan (young kid profile, no login — MVP) | None | None | None | None | None | None | None | None | None | None | None | None | None | None | None | None | None | None |
| Jordan (older kid, limited login — Later) | None | View | None | None | None | Full | View | Own-only | None | None | None | None | None | None | None | None | Own-only | None |
| Riley (Operator, support — from v1) | View | View | None | None | View | View | View | View | None | None | None | View | None | None | View | None | None | Full |

Notes on the matrix:

- **Young kid profiles** have no login, so every cell is None; their dietary rules are protected through Maya's Household Setup access, and Maya or Sam record a young kid's post-dinner rating on the kid's behalf (Meal Rating & Preference Learning).
- **Sam's View on Weekly Plan** still lets him mark a leftover lunch as eaten or skipped — a status update, not a change to the plan.
- **Maya's Support View** is View of the record showing when support viewed the household and why; Riley's Full is full use of a read-only view that has no edit controls.
- **Riley's Billing View** covers the plan tier only, never payment details; **Riley's Kid Profile Data** is None except the allergy details inside a specific safety report.
- **The older-kid row's Grocery List Full** covers adding and ticking items, not household list settings (aisle names sit under Household Setup) or grocery-ordering handoff.
- **Unauthorized visitors** (anyone not signed in as a member of the household) see only a sign-in screen, an invitation-acceptance screen for a link addressed to them, or the public welcome page of a household referral link — never household data.
