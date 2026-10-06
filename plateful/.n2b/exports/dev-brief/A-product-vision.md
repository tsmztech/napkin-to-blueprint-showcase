# Part A — Product Vision

This part establishes what Plateful is and who it is for: the executive summary, the founder's complete brief, and the persona set with its access matrix. Read it before any other part.

## Executive Summary

Plateful is a shared family meal planner: a household is set up once with who eats together, what each person cannot or will not eat, the weeknight time available and the weekly food budget, and the product then proposes a realistic weekly dinner plan, a swap on every meal, and one shared, aisle-grouped grocery list, treating allergies as a hard rule checked by the app itself. It serves household organisers, other adult members and, through parent-managed profiles, kids. This package specifies 25 features in 196 specifications carrying 2292 acceptance criteria, a recommended architecture with documented alternatives, and a complete database schema.

## The Brief

The founder's brief follows in full, including its open questions (which are also surfaced in Part C).


# Plateful

## Vision
Plateful is a shared family meal planner. The household is set up once: who eats with us, what each person can't or won't eat, how much time there is on which nights, and the weekly food budget. After that, AI proposes a realistic 7-day dinner plan every week that respects all of it. Any meal can be swapped with one tap, and the plan uses up what the family says is already in the fridge. It produces one combined grocery list, grouped by supermarket aisle, that the whole household shares and ticks off live in the store. Leftovers roll into lunches, and over time the plan learns what the family actually likes. Allergies are a hard rule: the app itself checks every suggestion against the household's allergy list, so the AI is never the last line of defence.

## Problem Statement
Every Sunday night families face the same question: what are we eating this week? They have to juggle a child's food allergy, a partner's vegetarian diet, a picky eater and 30-minute weeknights. Today they plan with recipe screenshots, a notes app, a half-remembered fridge and a grocery list that three people edit in a group chat. They still end up with takeaway mid-week and food in the bin by Saturday. The existing options fall short. Recipe sites are stuffed with ads, and meal-kit companies want to sell boxes. "AI meal plan" tools ignore real constraints: one suggests a walnut salad to a family with a nut allergy.

## Target Users & Roles
- **Household organiser** (usually one parent). A working parent with two kids, one of whom has a food allergy, and a partner on a different diet. They have 30-minute weeknights and currently plan in a notes app and a group chat. They create the household, invite others, and set everyone's dietary rules, the budget and the weekly schedule. They also approve the weekly plan. They reach for Plateful when the Sunday "next week's plan is ready" notification arrives. They may use a bigger screen for the initial setup.
- **Other adult members** (partner, roommate, a grandparent who cooks on Tuesdays). They see the plan and suggest swaps. They add to and tick off the grocery list and rate meals. They reach for it in the supermarket with the shared list open, at the evening "tonight's dinner" nudge, and when rating meals after dinner.
- **Kids.** Their allergies and dislikes drive the plan, and older kids want to vote on dinners. Young kids should not need their own account. Exactly how kids are represented is an open question. The founder leans towards parent-managed profiles with no login for young kids, and possibly a limited login for older kids later. Kids' data is kept minimal.
- **Operator support (not a product role).** The founder, as operator, needs only read-only support access to help a household. Nothing more.

Roles were confirmed in conversation: organiser, other adult members and kids. There is one household per account in v1, and everyone in a household sees the same plan and grocery list.

## The Experience
It's Sunday evening and a notification arrives: "Next week's plan is ready." You open it on your phone and see seven dinners. Each shows cook time, rough cost and small badges confirming it's nut-free, checked against your allergies, and has a vegetarian option. Thursday says "uses the spinach and feta you already have". You swap Monday's curry for something faster with one tap, and the grocery list updates instantly. On Saturday your partner is in the supermarket with the same list open, ticking things off. The teenager adds "more yoghurt" from the couch and it appears on the list in real time. On Wednesday at 5pm a nudge arrives: "Tonight: 20-minute pasta — take the chicken out of the freezer." After dinner the kids tap a thumbs up or down. The key moment comes in the first week: nothing gets thrown away, and nobody asks "what's for dinner?"

## Business Context
Plateful is a commercial product for families. The free tier covers manual planning and the shared grocery list. A paid household subscription (monthly or yearly) adds the AI weekly plan, pantry-aware suggestions and learning from ratings. There are no ads, ever, and no selling of family data. Parents are sensitive about this, so it is a selling point. Why now: AI can finally plan around real constraints, higher food prices make waste hurt, and every family already shares a grocery list somewhere awful. The first users will be parents from the founder's kids' school and online parenting groups. Growth is expected to come mostly from households inviting other households.

## Scale & Non-Functional Expectations
- **Order of magnitude:** several thousand households in the first year, with 2–6 people each.
- **Devices / platforms:** a mobile-first responsive web app. People plan on the sofa and shop with the phone in one hand, and the organiser may use a laptop for setup. There are no native apps in v1.
- **Geography:** the US and UK first. Units (cups vs grams), currency and supermarket aisle names must be configurable, not hard-coded.
- **Performance / availability:** the shared grocery list must feel instant and must keep working in a supermarket with bad signal. Changes made offline sync later. No specific uptime number was stated.
- **Cost:** the AI cost per household must stay small, roughly one weekly plan plus a few swaps. The free tier gets no AI.
- **Privacy:** children's data is minimal, parent-controlled and never used for anything but the family's own plan. There are no ads and no data selling.

## Ecosystem & Integrations
- **AI model:** generates the weekly plans and suggestions. The founder has no preference on which one.
- **Recipe sources:** a starter recipe library, plus saving recipes from any website by pasting a link. Whether importing recipes from other sites is legal is an open question.
- **Online grocery ordering (e.g. Instacart, Tesco):** desirable later, not v1.
- **Family calendar:** a nice-to-have for showing dinner on the family calendar, not v1.
- Otherwise the product stands alone (confirmed).

## Success Criteria
- Families say they throw away noticeably less food and spend less.
- The organiser spends under 10 minutes a week on planning.
- Most paying households were invited by another household.
- It has never once suggested a meal that breaks a family member's allergy.

## Constraints
- **Team / timeline:** the founder is building solo with AI coding tools and wants paying households within about three months. *Rationale:* a one-person venture that needs early revenue.
- **Budget:** infrastructure stays under roughly $100/month until there's revenue, so AI costs per household must stay small. *Rationale:* pre-revenue bootstrapping.
- **Safety (allergies):** allergies are a hard rule, not a preference. The AI must never be the last line of defence: the app checks every suggestion against the household's allergy list before anyone sees it. Meals show a "checked against allergies" badge with a standard "always check labels" disclaimer. *Rationale:* a single unsafe suggestion could harm a child and destroy trust.
- **Dietary rule strength:** allergies and religious rules (e.g. halal) are hard filters. Vegetarian can be set per person, with a "vegetarian option" on a shared meal. Dislikes are soft and learned from ratings. *Rationale:* stated by the founder when confirming how the household's rules apply.
- **Privacy / children (regulatory):** children use the product, so their data is minimal, parent-controlled and never used for anything but the family's own plan. There are no ads and no data selling. There is no medical or diet advice. *Rationale:* parents' sensitivity, and this is a selling point.
- **Platform:** a responsive web app for v1, with no native apps and no app stores. *Rationale:* a solo founder shipping fast.
- **Technology:** no technology commitments. Build it with whatever the blueprint recommends.
- **Design preference (no design system supplied):** friendly, calm and food-photo-led, with big tap targets for one-handed use in a shop. *Rationale:* people plan on the sofa and shop with the phone in one hand.

## Open Questions
- **Kids' representation:** should kids be profiles managed by a parent, or have their own login once they're old enough? At what age? The founder leans towards parent-managed profiles with no login for young kids, and possibly a limited login for older kids later.
- **Pantry depth:** should Plateful track a detailed pantry inventory, or just "use up what I tell you I have"?
- **Meal scope:** breakfast and lunch too, or dinners only for v1? Leftovers rolling into lunches is already part of the vision.
- **Recipe import legality:** is saving recipes from other websites legally OK, and in what form?
- **Nutrition information:** are calories and macros useful, or a liability for families? No medical or diet advice is intended either way.

## Source Materials
- `.n2b/inputs/source/pasted-notes.md` — the founder's full written idea (problem, product, users, experience, business, scale, integrations, success, hard boundaries, open questions), pasted at intake.

*Preserved verbatim so nothing the user wrote is lost in this brief's compression. Stage 2 works from this brief (brief-first); the originals are kept for reference and for Stage 1 re-runs.*


## Who This Is For

The persona set follows: primary and secondary personas and the role access matrix.


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
