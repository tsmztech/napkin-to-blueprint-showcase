---
document_type: user-persona
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# User Personas

## Persona Set Summary

This product serves three distinct household roles plus one non-product support role. The primary persona is the household organiser, who sets up the household and approves the weekly plan (BRIEF.md, Target Users & Roles). Two secondary personas — other adult household members and kid profiles — reflect the brief's explicit description of a shared household plan and grocery list used by everyone in the household. A fourth, lightweight entry captures the founder's read-only operator support role, which the brief itself names but explicitly marks "not a product role."

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
- Approve or adjust the week's plan quickly when it does not quite fit (BRIEF.md, The Experience)

### Pain Points

- Sunday-night planning is a recurring chore that has to juggle an allergy, a partner's diet, a picky eater, and 30-minute weeknights all at once (BRIEF.md, Problem Statement)
- Existing "AI meal plan" tools ignore real constraints — one suggested a walnut salad to a family with a nut allergy (BRIEF.md, Problem Statement)
- Recipe sites are ad-stuffed and meal-kit companies are trying to sell boxes, not solve the actual planning problem (BRIEF.md, Problem Statement)
- The current grocery list lives in a group chat that three people edit, which is unreliable and easy to lose track of (BRIEF.md, Problem Statement)
- Despite all the planning effort, the household still ends up with mid-week takeaway and food in the bin by Saturday (BRIEF.md, Problem Statement)

### Behavioral Context

Maya's main touchpoint is a Sunday-evening notification that the next week's plan is ready, which she opens on her phone from the sofa. She may use a bigger screen (a laptop) for the initial household setup, since that involves more detailed input (members, allergies, budget, schedule), but day-to-day interaction — reviewing the plan, swapping a meal, approving changes — happens on her phone in short sessions (BRIEF.md, Target Users & Roles, Scale & Non-Functional Expectations).

### What This User Does NOT Need

- A detailed nutrition or calorie-tracking tool — the brief explicitly states no medical or diet advice is intended (BRIEF.md, Open Questions, Constraints)
- Native mobile apps — v1 is a responsive web app only (BRIEF.md, Constraints: Platform)
- Meal-kit box ordering or ads inside the product — the brief positions Plateful against both (BRIEF.md, Problem Statement, Business Context)
- Multi-household management — there is one household per account in v1 (BRIEF.md, Target Users & Roles)

## Secondary Personas

### Other Adult Member

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section names "other adult members (partner, roommate, a grandparent who cooks on Tuesdays)" as people who see the plan, suggest swaps, tick off the grocery list, and rate meals — a distinct entitlement set from the organiser's setup-and-approve role.

**Name:** Sam

**Description:** Sam is Maya's partner (or, in another household, a roommate or a grandparent who cooks regularly). Sam follows a different diet than the rest of the household and shares in cooking and shopping duties, but did not set up the household and does not manage its dietary rules or budget.

**Goals:** See the week's plan without having to ask; top up the shared grocery list from wherever they are; tick items off while actually in the supermarket, even with a weak signal; rate meals honestly after dinner so future plans improve.

**Pain Points:** The current grocery list is a group chat three people edit, which is easy to lose track of (BRIEF.md, Problem Statement); Sam has no reliable way today to see what's already been decided for the week without checking with Maya.

**Behavioral Context:** Sam opens the app with the shared list already loaded while standing in the supermarket, one hand on a trolley; also uses it at the evening "tonight's dinner" nudge and right after dinner to rate the meal (BRIEF.md, The Experience).

**What This User Does NOT Need:** Household setup screens, budget configuration, or dietary-rule administration — those belong to the organiser (BRIEF.md, Target Users & Roles).

### Kid Profile

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section states "kids' allergies and dislikes drive the plan, and older kids want to vote on dinners," and that young kids should not need their own account, with parent-managed profiles as the founder's leaning; this is a genuinely distinct entitlement set (little or no login, no setup access) from either adult role.

**Name:** Jordan

**Description:** Jordan is one of Maya's kids. As a younger child, Jordan has no login at all — Maya manages Jordan's allergy and dislike profile on their behalf. As an older kid (a distinct sub-case the brief leaves open for a possible future limited login), Jordan might vote on dinners and rate meals directly.

**Goals:** Have their allergy and dislikes genuinely respected in every suggested meal; as an older kid, have a say in which dinner gets picked; add small things to the shared list ("more yoghurt") without needing an account.

**Pain Points:** Existing "AI meal plan" tools do not reliably respect a stated allergy, which is exactly the risk this persona embodies most directly (BRIEF.md, Problem Statement).

**Behavioral Context:** For young kids, all interaction is indirect — Maya enters and maintains Jordan's allergy/dislike data. For an older kid with a limited login (a Later-phase possibility), interaction is occasional: voting on the week's dinners, adding an item to the list from the couch, and a post-dinner thumbs up/down (BRIEF.md, The Experience, Target Users & Roles).

**What This User Does NOT Need:** A full account, household setup access, budget or dietary-rule administration, or any login at all as a young child — the founder's stated default is parent-managed profiles with no login for young kids (BRIEF.md, Target Users & Roles, Open Questions).

### Operator (Support)

**Provenance:** [INFERRED] — BRIEF.md's Target Users & Roles section names "Operator support (not a product role)" explicitly, stating "the founder, as operator, needs only read-only support access to help a household. Nothing more." This is included here, lightly, purely because the brief names it directly and grounded-roles requires every access level in the product to trace to a cited source — it is not part of the product's user-facing role model.

**Name:** Riley

**Description:** Riley is the founder acting in a support capacity, occasionally helping a household troubleshoot a problem with their plan or list.

**Goals:** See enough of a household's setup and plan to diagnose a reported problem, without being able to change anything or see more than necessary.

**Pain Points:** N/A — this is a support capability, not a persona with product pain points of their own.

**Behavioral Context:** Rare, on-demand use triggered by a support request; never a routine or scheduled interaction.

**What This User Does NOT Need:** Any ability to edit household data, view billing details beyond plan tier, or see children's dietary data beyond what is strictly needed to diagnose an allergy-safety report — the brief's privacy constraint on children's data applies here too (BRIEF.md, Constraints: Privacy).

## Access Matrix

| Role / Persona | Household Setup | Weekly Plan | Meal Swap | Pantry Input | Grocery List | Recipe Library | Ratings | Notification Prefs | Billing | Kid Profile Data |
|---|---|---|---|---|---|---|---|---|---|---|
| Maya (Organiser) | Full | Full | Full | Full | Full | Full | Full | Full | Full | Full |
| Sam (Other Adult Member) | View | View | Full | Full | Full | Full | Own-only | Own-only | None | View |
| Jordan (Kid Profile) | None | View | None | None | Full | View | Own-only | None | None | None |
| Riley (Operator, support) | View | View | None | View | View | View | View | None | View | None |
