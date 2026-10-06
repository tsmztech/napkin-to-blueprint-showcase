---
document_type: success-metrics
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# Success Metrics

## Summary

This document contains 11 success metrics covering all 9 Core features plus 2 Important features: 4 product-experience metrics, 5 adoption/engagement/business KPIs, and 2 user-facing performance expectations. Together they track whether Plateful actually reduces food waste and planning time, whether growth comes from household invitations as the business model expects, and whether the shared list and plan feel fast and trustworthy in daily use.

---

### Weekly Planning Time

**Description:** Measures how much time the organiser spends on weekly planning, from opening the plan-ready notification to having a plan they're satisfied with for the week.

**Target:** The organiser reaches a plan they're satisfied with — including any swaps — in under 10 minutes per week.

**Rationale:** BRIEF.md's Success Criteria states directly: "The organiser spends under 10 minutes a week on planning." This is the product's headline time-saving promise.

**Persona:** Maya

**Connected Feature:** AI Weekly Dinner Plan Generation

---

### Zero Allergy Incidents

**Description:** Measures whether any AI-suggested meal that reaches a household ever violates a stated allergy.

**Target:** Zero instances, ever, of a suggested meal violating any household member's stated allergy, across all households.

**Rationale:** BRIEF.md's Success Criteria states: "It has never once suggested a meal that breaks a family member's allergy." This is a non-negotiable trust metric, not a rate to be minimized — it is a hard zero.

**Persona:** All

**Connected Feature:** Dietary Rules & Allergy Safety Engine

---

### Reported Food Waste Reduction

**Description:** Measures whether households report throwing away less food after adopting Plateful, tying directly to the pantry-aware plan and leftover rollover.

**Target:** At least 3 in 4 active households report throwing away noticeably less food after their first month of regular use, compared to before.

**Rationale:** BRIEF.md's Success Criteria states: "Families say they throw away noticeably less food and spend less." This is the product's core value proposition made measurable.

**Persona:** All

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

**Description:** Measures whether household members experience the shared grocery list as genuinely live and reliable, including through supermarket connectivity gaps.

**Target:** A tick or manual add made by one household member is visible to another household member within 2 seconds under normal connectivity, and every offline change syncs correctly with no duplicates once connectivity returns.

**Rationale:** BRIEF.md's Scale & Non-Functional Expectations states the list "must feel instant and must keep working in a supermarket with bad signal." This is the mechanism that replaces the unreliable group-chat list described in the Problem Statement.

**Persona:** All

**Connected Feature:** Shared Grocery List

---

### Weekly Plan Ready Notification Reach

**Description:** Measures whether the Sunday "plan ready" notification actually reaches and is opened by household members who enabled it.

**Target:** At least 80% of enabled notifications are delivered within one minute of plan generation completing, and at least half are opened within the same day.

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

### Household-to-Household Invitation Growth

**Description:** Measures whether new paying households are arriving primarily through invitations from existing households, as the brief's growth model expects.

**Target:** At least half of new paying households in a given month were invited by another household member of an already-active household, rather than arriving through other channels.

**Rationale:** BRIEF.md's Success Criteria states: "Most paying households were invited by another household." BRIEF.md's Business Context also names this as the expected primary growth channel.

**Persona:** All

**Connected Feature:** Household Invitations & Membership

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

### Weeknight Time-Fit Accuracy

**Description:** Measures whether dinners suggested for time-constrained weeknights actually fit the household's stated available time.

**Target:** At least 95% of dinners planned for a night the household marked as time-constrained (e.g., a 30-minute weeknight) have a stated cook time at or under that limit.

**Rationale:** BRIEF.md's Problem Statement names "30-minute weeknights" as one of the core constraints families juggle today; a plan that ignores this defeats the product's central promise of respecting real household constraints.

**Persona:** All

**Connected Feature:** AI Weekly Dinner Plan Generation
