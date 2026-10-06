---
project_name: Plateful
domain: household meal planning and food waste
created: 2026-09-26
status: active
n2b_version: 0.4.0
---

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
