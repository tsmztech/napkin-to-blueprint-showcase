---
document_type: success-metrics
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed (1 fix applied)
---

# Success Metrics

## Summary

This document contains 15 success metrics covering all 10 Core features plus 3 Important features: 8 product-experience metrics, 5 adoption/engagement/business KPIs, and 2 user-facing performance expectations. Together they track whether Plateful actually reduces food waste, spend, and planning time, whether allergy safety holds without exception, whether growth comes from households inviting other households as the business model expects, and whether the shared list and plan feel fast and trustworthy in daily use. [MODIFIED: 4 metrics added (Household Member Participation, Manual Week Completion, Pantry Items Used, Paying Household Retention) so every Core feature — including Manual Weekly Planning added in synthesis — has its own metric and the KPI set includes retention]

---

### Weekly Planning Time

**Description:** Measures how much time the organiser spends on weekly planning, from opening the plan-ready notification to having a plan they're satisfied with for the week.

**Target:** The organiser reaches a plan they're satisfied with — including any swaps and answering other adults' swap suggestions — and approves it in under 10 minutes per week. [MODIFIED: target now includes reviewing suggestions and approving, the steps the organiser owns under BRIEF.md's role split]

**Rationale:** BRIEF.md's Success Criteria states directly: "The organiser spends under 10 minutes a week on planning." This is the product's headline time-saving promise.

**Persona:** Maya

**Connected Feature:** AI Weekly Dinner Plan Generation

---

### Zero Allergy Incidents

**Description:** Measures whether any meal that reaches a household's plan — AI-suggested, swapped in, or picked by hand — ever violates a stated allergy.

**Target:** Zero instances, ever, of a meal on any household's plan violating any household member's stated allergy, across all households; every safety concern a household reports is acted on (the meal leaves the plan) at the moment it is reported. [MODIFIED: scope widened from AI suggestions to every path onto the plan, and the safety-concern report added, matching the final Dietary Rules & Allergy Safety Engine]

**Rationale:** BRIEF.md's Success Criteria states: "It has never once suggested a meal that breaks a family member's allergy." This is a non-negotiable trust metric, not a rate to be minimized — it is a hard zero. [RESEARCH-INFORMED: no competitor enforces an app-level allergy check, and a publicized AI meal planner produced dangerous recipes (market research, HIGH and MEDIUM confidence) — the zero is Plateful's differentiator]

**Persona:** All

**Connected Feature:** Dietary Rules & Allergy Safety Engine

---

### Reported Food Waste and Spend Reduction

**Description:** Measures whether households report throwing away less food and spending less after adopting Plateful, tying directly to the pantry-aware plan and leftover rollover, using the household's own weekly check-in answers compared with its starting point.

**Target:** At least 3 in 4 active households that answer the check-in report throwing away noticeably less food after their first month of regular use, compared to their starting point, and at least half report grocery spend at or under their weekly budget in 3 of their last 4 answered weeks. [MODIFIED: "spend less" half of the brief's criterion added, and the measure tied to the Weekly Waste & Spend Check-In that now captures these answers]

**Rationale:** BRIEF.md's Success Criteria states: "Families say they throw away noticeably less food and spend less." This is the product's core value proposition made measurable.

**Persona:** All

**Connected Feature:** Weekly Waste & Spend Check-In

---

### Pantry Items Used

**Description:** Measures whether the things a household says it already has actually get used up by the plan rather than forgotten.

**Target:** Of the pantry items a paid household logs before its weekly plan is generated, at least 60% appear in a dinner that week, and the household sees each one called out on the dinner that uses it. [AUDIT-ADDED: 1 -- Pantry-Aware Suggestions is Core and lost its own metric when the waste metric moved to the check-in]

**Rationale:** BRIEF.md's Vision promises "the plan uses up what the family says is already in the fridge," and The Experience shows "uses the spinach and feta you already have." [RESEARCH-INFORMED: the only competitor with pantry awareness is described as "basic" even on its paid tier (Samsung Food, MEDIUM confidence), so doing this visibly well is a differentiator]

**Persona:** Maya

**Connected Feature:** Pantry-Aware Suggestions

---

### One-Tap Swap Completion

**Description:** Measures whether swapping a meal is genuinely fast and low-friction, matching the brief's "one tap" promise.

**Target:** The user completes a meal swap — from tapping swap to seeing the plan and list updated — in under 10 seconds, in a single interaction.

**Rationale:** BRIEF.md's Vision states plans can be swapped "with one tap" and the list "updates instantly." If a swap requires multiple steps or a visible delay, the promise is not being met.

**Persona:** All

**Connected Feature:** One-Tap Meal Swap

---

### Grocery List Live-Update Trust

**Description:** Measures whether household members experience the shared grocery list as genuinely live and reliable, including through supermarket connectivity gaps, and whether every member always sees the same plan and the same list.

**Target:** A tick or manual add made by one household member is visible to another household member within 2 seconds under normal connectivity, every offline change syncs correctly with no duplicates once connectivity returns, and no household member ever sees a week's plan without its matching list. [RESEARCH-INFORMED: the one competitor combining AI plans with household sharing reports members seeing the plan but not the list (Samsung Food, MEDIUM confidence)]

**Rationale:** BRIEF.md's Scale & Non-Functional Expectations states the list "must feel instant and must keep working in a supermarket with bad signal." This is the mechanism that replaces the unreliable group-chat list described in the Problem Statement.

**Persona:** All

**Connected Feature:** Shared Grocery List

---

### Weekly Plan Ready Notification Reach

**Description:** Measures whether the "plan ready" message actually reaches and is opened by household members who enabled it.

**Target:** At least 80% of enabled plan-ready messages (device notification, or email where device notifications are unavailable) are delivered within one minute of plan generation completing, and at least half are opened within the same day. [MODIFIED: email fallback included, matching the final Weekly Plan Ready Notification]

**Rationale:** BRIEF.md's own description of the core experience begins with this notification arriving; if it doesn't reach people reliably, the whole weekly rhythm the product depends on breaks down.

**Persona:** All

**Connected Feature:** Weekly Plan Ready Notification

---

### Recipe Library Coverage at Launch

**Description:** Measures whether the starter recipe library gives a brand-new household, with no imported recipes yet, enough variety to generate a genuinely varied first week.

**Target:** A new household with typical dietary rules (e.g., one allergy, one vegetarian member) receives a first-week plan with no repeated dinners and at least one recipe per major cuisine style represented in the library.

**Rationale:** Without adequate starter coverage, the very first plan — the moment that sets first impressions — could feel thin or repetitive before any recipe import happens.

**Persona:** Maya

**Connected Feature:** Recipe Library (Starter Recipes)

---

### Manual Week Completion

**Description:** Measures whether a free-tier organiser can plan a week by hand and get a usable grocery list without giving up partway.

**Target:** At least 60% of free-tier organisers who start planning a week by hand fill at least five nights and see the grocery list built from them in the same session. [AUDIT-ADDED: 2 -- metric for Manual Weekly Planning, a Core feature added in synthesis]

**Rationale:** BRIEF.md's Business Context makes manual planning the free tier's core offer; every new household starts there, so a manual week that feels like hard work would lose households before they ever see the paid plan.

**Persona:** Maya

**Connected Feature:** Manual Weekly Planning

---

### Household Member Participation

**Description:** Measures whether households actually become shared — other adults joining and using the plan and list — rather than staying a single organiser's tool.

**Target:** At least 60% of households that have been active for two weeks have at least one other adult member who has joined and ticked, added, or suggested something in that time. [AUDIT-ADDED: 1 -- Household Invitations & Membership is Core and needed its own metric once household-to-household growth moved to Invite Another Household]

**Rationale:** BRIEF.md's Vision describes a list "the whole household shares and ticks off live in the store," and Target Users & Roles states "everyone in a household sees the same plan and grocery list"; a household nobody else joins cannot deliver that.

**Persona:** Sam

**Connected Feature:** Household Invitations & Membership

---

### Household-to-Household Invitation Growth

**Description:** Measures whether new paying households are arriving primarily through invitations from existing households, as the brief's growth model expects.

**Target:** At least half of new paying households in a given month set up their household from another household's invite link, rather than arriving through other channels.

**Rationale:** BRIEF.md's Success Criteria states: "Most paying households were invited by another household." BRIEF.md's Business Context also names this as the expected primary growth channel. [MODIFIED: synthesis check 9 — reconnected from Household Invitations & Membership, which adds people within one household, to Invite Another Household, the feature that records household-to-household referrals]

**Persona:** All

**Connected Feature:** Invite Another Household

---

### First-Session Onboarding Completion

**Description:** Measures whether an organiser can go from account creation to a fully usable household setup in one sitting, without needing to return later to finish.

**Target:** At least 70% of organisers who start household setup complete it — members, dietary rules, budget, and schedule — within their first session.

**Rationale:** The founder is building solo and needs paying households within about three months (BRIEF.md, Constraints: Team/timeline); a setup flow that loses people midway directly threatens that timeline.

**Persona:** Maya

**Connected Feature:** Household Setup & Member Profiles

---

### Paid Conversion Rate

**Description:** Measures whether households on the free tier convert to the paid subscription that unlocks the AI weekly plan.

**Target:** At least 1 in 5 households active on the free tier for two or more weeks upgrade to a paid subscription.

**Rationale:** BRIEF.md's Business Context describes the paid tier as the product's revenue mechanism, and the founder needs "paying households within about three months" (BRIEF.md, Constraints: Team/timeline) — this metric is the direct measure of that goal.

**Persona:** Maya

**Connected Feature:** Subscription & Billing Management

---

### Paying Household Retention

**Description:** Measures whether households that pay keep paying and keep using the weekly plan, rather than trying it once and leaving.

**Target:** At least 70% of households that upgrade are still paying and still opening their weekly plan three months later. [AUDIT-ADDED: 1 -- the adoption/engagement KPI set had conversion but no retention measure]

**Rationale:** BRIEF.md's Constraints: Team/timeline and Budget describe a one-person, pre-revenue venture; revenue only holds if households stay. [RESEARCH-INFORMED: three meal-planning apps closed or were folded into other products within about two years (market research, MEDIUM confidence), so durable household retention is the test a subscription planner must pass]

**Persona:** Maya

**Connected Feature:** Subscription & Billing Management

---

### Weeknight Time-Fit Accuracy

**Description:** Measures whether dinners suggested for time-constrained weeknights actually fit the household's stated available time.

**Target:** At least 95% of dinners planned for a night the household marked as time-constrained (e.g., a 30-minute weeknight) have a stated cook time at or under that limit.

**Rationale:** BRIEF.md's Problem Statement names "30-minute weeknights" as one of the core constraints families juggle today; a plan that ignores this defeats the product's central promise of respecting real household constraints.

**Persona:** All

**Connected Feature:** AI Weekly Dinner Plan Generation
