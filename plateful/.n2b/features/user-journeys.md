---
document_type: user-journeys
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-26
synthesis_check: passed (4 fixes applied)
---

# User Journeys

## Journey

This document contains 10 journeys covering the household's full lifecycle: first setup, the weekly planning loop (AI-generated on the paid tier and hand-picked on the free tier), shopping, weeknight dinners, recipe and pantry management, allergy-safe recovery, joining a household, account and billing management, and the end-of-week check-in that brings in other families. Journeys are owned by Maya (organiser) and Sam (other adult member) — the two roles with their own logins in MVP; Jordan (kid profile) appears inside journeys through the rules and ratings adults record for them. Every Core and Important feature appears across the set, and all three coverage types (First-use, Regular, Edge/Recovery) are represented. [MODIFIED: two journeys added (Free-Tier Manual Week; End-of-Week Check-In & Inviting Another Family) so that the three features added in synthesis — Manual Weekly Planning, Invite Another Household, Weekly Waste & Spend Check-In — each appear in a journey]

---

### First Household Setup

**Owning Persona:** Maya

**Coverage:** First-use

**Journey Goal:** Get the household fully set up — members, dietary rules, budget, and schedule — so the first week of dinners can be planned.

**Entry Point:** Maya creates a Plateful account for the first time, motivated by wanting to stop re-planning dinner from scratch every Sunday.

**Steps:**

1. Create the household — Maya signs up, names the household and becomes its first member. She sees a short, guided setup rather than a long form.
2. Add members and rules — Maya adds her partner and two kids, entering each person's allergies, diets, and dislikes, including her child's nut allergy as a hard rule picked from the standard allergen list. For each kid profile she confirms she is the parent and sees that only a first name, an age band, and dietary rules are stored. [AUDIT-ADDED: 1 -- journey walk: standard allergen list and parental-consent confirmation added to Household Setup & Member Profiles]
3. Set the practical facts — Maya states the weekly food budget and marks which weeknights are short on time.
4. Set units, currency and aisle layout — Maya confirms her local units, currency, and default supermarket aisle groupings.
5. Invite her partner — Maya sends her partner an invitation so he will see the same plan and list. [MODIFIED: synthesis check 14 — Household Invitations & Membership was listed as a connected feature but no step used it]
6. Setup complete — Maya sees a clear confirmation that the household is ready and a choice of what comes next: pick this week's dinners herself on the free tier, or upgrade so the AI plan arrives on Sunday evening. [MODIFIED: synthesis check 3 — the draft promised every new household an AI plan, but new households start on the free tier, which has no AI (BRIEF.md, Business Context)]

**Failure/Recovery Variant:** Maya is interrupted partway through entering member dietary rules and closes the app. When she reopens Plateful later, setup resumes exactly where she left off — her partner's and first child's rules are preserved, and she is not asked to re-enter anything, consistent with FEAT-01's partial-setup handling.

**Success Outcome:** Within one sitting, the household exists with accurate allergy and diet data, a budget, a schedule, and locale settings — everything a safe, realistic first week needs, whether Maya picks the dinners herself or the AI proposes them.

**Connected Features:** Household Setup & Member Profiles, Dietary Rules & Allergy Safety Engine, Household Invitations & Membership, Units, Currency & Locale Configuration, Manual Weekly Planning, Subscription & Billing Management

---

### Sunday Plan Review & First Swap

**Owning Persona:** Maya

**Coverage:** Regular

**Journey Goal:** Review the week's AI-generated plan, adjust one meal that doesn't fit, and approve it so the household can see the final week.

**Entry Point:** A Sunday-evening notification arrives: "Next week's plan is ready."

**Steps:**

1. Open the notification — Maya taps it and lands directly on the new week's plan.
2. Scan the week — She sees seven dinners, each with cook time, rough cost, and a "checked against allergies" badge, plus the week's estimated total against the budget; Thursday calls out that it uses spinach and feta already in the fridge. [RESEARCH-INFORMED: weekly total against budget added with the budget capability of AI Weekly Dinner Plan Generation]
3. Check household input — Sam has suggested swapping Friday's fish for a family favourite; Maya accepts it with one tap. In households where older-kid voting is enabled (a Later-phase capability), she would also see the older kid's vote on one night's options here. [MODIFIED: synthesis check 3 — the draft showed an older-kid vote as present in every Sunday review, but Older-Kid Dinner Voting is Later-phase; the MVP step now shows the other-adult swap suggestion that BRIEF.md, Target Users & Roles describes ("suggest swaps")]
4. Swap a meal — Monday's curry looks too slow for a busy night, so she taps swap and picks a faster, already-safety-checked alternative in one tap.
5. Approve and share — Maya approves the week; the plan and grocery list are already up to date, and the rest of the household sees the finalized week. [AUDIT-ADDED: 1 -- journey walk: BRIEF.md states the organiser approves the weekly plan]

**Failure/Recovery Variant:** After Maya swaps Monday's meal, the swap fails to save due to a dropped connection. The original meal remains visible rather than an empty slot, and Maya is offered an immediate retry; the second attempt succeeds and the list updates correctly.

**Success Outcome:** The week's plan reflects the household's real schedule and budget with no unsafe suggestions, and the whole household can see it — all within a few minutes of the notification arriving.

**Connected Features:** AI Weekly Dinner Plan Generation, One-Tap Meal Swap, Weekly Plan Ready Notification, Dietary Rules & Allergy Safety Engine, Older-Kid Dinner Voting, Shared Grocery List

---

### Supermarket Shared Shopping

**Owning Persona:** Sam

**Coverage:** Regular

**Journey Goal:** Do the week's shopping efficiently using the shared, aisle-grouped grocery list, even with a weak in-store signal.

**Entry Point:** Sam arrives at the supermarket on Saturday with the shared grocery list already open on their phone.

**Steps:**

1. Open the list — Sam sees the combined list, grouped by aisle, generated automatically from the week's plan, with ingredients needed by several dinners combined into one line. [RESEARCH-INFORMED: ingredient consolidation across recipes is praised as a time-saver in competitor reviews (Samsung Food, AnyList)]
2. Shop by aisle — Sam moves through the store ticking off items as they go; each tick is visible to the rest of the household in real time.
3. Add on the fly — Sam remembers the household is low on olive oil and adds it directly to the list mid-shop.
4. Lose signal briefly — In a corner of the store with no signal, Sam keeps ticking items off; the app holds the changes locally.
5. Regain signal and finish — Once back near the entrance, Sam's offline ticks sync automatically with no duplicates, and the list shows fully up to date.

**Failure/Recovery Variant:** While offline, Sam accidentally ticks the same item twice in quick succession. When the changes sync, the list correctly shows it ticked once, not double-counted or duplicated — the sync logic merges the offline changes cleanly.

**Success Outcome:** Sam completes the week's shopping from one shared, correctly organized list, without needing to double-check anything with Maya, even through a stretch of poor signal.

**Connected Features:** Shared Grocery List, Units, Currency & Locale Configuration

---

### Weeknight Dinner, Nudge & Rating

**Owning Persona:** Sam

**Coverage:** Regular

**Journey Goal:** Get reminded what's for dinner, cook it with the right prep already done, and let the household's ratings feed future plans.

**Entry Point:** A 5pm notification arrives: "Tonight: 20-minute pasta — take the chicken out of the freezer."

**Steps:**

1. Receive the nudge — Sam sees the dinner nudge and the reminder to take the chicken out of the freezer, prepared in time for cooking.
2. Cook the planned dinner — Sam prepares the 20-minute pasta using the plan, with the leftover-lunch note showing extra will be saved for tomorrow.
3. Rate after dinner — After eating, Sam gives a quick thumbs up or down, then passes his phone round the table so each kid taps a thumbs up or down that is recorded against their own kid profile. [MODIFIED: synthesis check 6 — young kid profiles have no login (BRIEF.md, Target Users & Roles), so kids' ratings are recorded on an adult's phone rather than "their own"]
4. Confirm the leftover lunch — The next day, Sam confirms the leftover pasta was eaten for lunch rather than thrown away.

**Failure/Recovery Variant:** The dinner is swapped to a different meal earlier that same afternoon. Sam already received the original 5pm nudge before the swap; a brief follow-up correction notification arrives naming the new dinner, so Sam is not caught out cooking the wrong thing.

**Success Outcome:** The household eats the planned dinner with the right prep done in advance, rates it honestly, and the leftovers become lunch instead of waste — closing the loop the brief opens with.

**Connected Features:** Tonight's Dinner Reminder, Leftover Rollover to Lunches, Meal Rating & Preference Learning, One-Tap Meal Swap

---

### Recipe Import & Pantry Update

**Owning Persona:** Maya

**Coverage:** Regular

**Journey Goal:** Add a recipe found online and update what's in the fridge, so next week's plan reflects both.

**Entry Point:** Maya finds a recipe on a food blog she wants to try and remembers she has odds and ends in the fridge to use up.

**Steps:**

1. Paste the link — Maya pastes the recipe's web link into Plateful.
2. Review extracted details — She checks the extracted ingredients and steps, correcting one quantity that came through oddly, then saves it.
3. See it in the library — The recipe now appears in the household's recipe library alongside the starter recipes, with its allergy badge already checked.
4. Log pantry items — Maya adds "half a bag of spinach, feta" to the pantry list before the next plan generates.
5. See it reflected — Next week's plan uses the imported recipe and calls out that it uses the logged spinach and feta.

**Failure/Recovery Variant:** The pasted link fails to extract cleanly (a page layout the parser can't read). Rather than a dead end, Maya is offered manual entry, types in the ingredients and steps herself, and the recipe saves successfully and appears in the library exactly like an imported one.

**Success Outcome:** Maya's found recipe and logged pantry items both make it into the household's actual planning, extending the recipe pool and reducing waste without extra effort.

**Connected Features:** Recipe Import from Web Link, Pantry-Aware Suggestions, Recipe Library (Starter Recipes)

---

### Allergy-Safe Swap Recovery

**Owning Persona:** Maya

**Coverage:** Edge/Recovery

**Journey Goal:** Swap a meal on a night when very few safe alternatives are available, and trust that nothing unsafe is ever shown.

**Entry Point:** Maya wants to swap Wednesday's dinner, but the household's combined dietary rules (a nut allergy, a vegetarian diet, and a dislike) narrow the field sharply.

**Steps:**

1. Tap swap — Maya requests alternatives for Wednesday's dinner.
2. See a short but safe list — Only two alternatives are offered instead of the usual several, each still fully checked against every household member's allergy and diet.
3. See the explanation — A short note explains that options are limited this week given the household's combined rules, rather than the app pretending there were more choices. [RESEARCH-INFORMED: multi-restriction households report sharply narrowed choices in preference-filtered planners (Mealime App Store reviews, MEDIUM confidence), so an honest explanation matters]
4. Pick one — Maya selects one of the two safe options, and the plan and grocery list update immediately.
5. Report a doubt — Looking at Thursday's dinner, Maya suspects a shop-bought sauce in it may contain nuts. She taps "report a safety concern"; the meal leaves the plan at once, she picks a safe replacement, and she is later told the outcome of the review. [AUDIT-ADDED: 1 -- journey walk error handling: households need a way to act when they doubt a badge; added to Dietary Rules & Allergy Safety Engine]

**Failure/Recovery Variant:** Maya first tries swapping into a recipe she remembers cooking before — but it isn't offered as an option, because its ingredient data can't be fully verified against her child's allergy. Rather than showing it and risking an unsafe suggestion, the app excludes it silently and explains, when she searches for it directly in the library, why it isn't eligible for her household's plan.

**Success Outcome:** Maya successfully swaps Wednesday's dinner, and at no point does an unsafe meal appear as an option — the allergy rule holds even under a highly constrained week, and a doubt about a meal is acted on immediately.

**Connected Features:** Dietary Rules & Allergy Safety Engine, One-Tap Meal Swap, Recipe Library (Starter Recipes)

---

### Invite & Join Household

**Owning Persona:** Sam

**Coverage:** First-use

**Journey Goal:** Accept an invitation to join Maya's household and immediately understand what's available.

**Entry Point:** Sam receives an invitation from Maya to join the household.

**Steps:**

1. Receive the invitation — Sam gets the invite and opens it.
2. Accept — Sam accepts, and a member profile is created for them as an Other Adult Member.
3. Land in context — Sam is taken directly to the current week's plan and grocery list, not a blank home screen.
4. Understand the role — A short explanation clarifies what Sam can do (view the plan, suggest swaps, shop, rate) versus what Maya manages (setup, budget, plan approval, billing). [MODIFIED: "swap meals" changed to "suggest swaps" and plan approval added to Maya's list, per BRIEF.md, Target Users & Roles]

**Failure/Recovery Variant:** Sam tries to accept the invitation two days after Maya revoked it by mistake (she meant to resend it). Sam sees a clear "this invitation is no longer valid" message rather than a confusing error, and Maya can send a fresh invitation once she notices.

**Success Outcome:** Sam is fully set up as a household member within a minute of accepting, immediately able to see and use the plan and list without needing Maya to walk them through it.

**Connected Features:** Household Invitations & Membership, Member Onboarding

---

### Upgrading & Managing the Account

**Owning Persona:** Maya

**Coverage:** Regular

**Journey Goal:** Upgrade the household to the paid tier to unlock the AI weekly plan, and later review or manage the household's own data.

**Entry Point:** Maya has been using the free tier's manual planning and shared list for a couple of weeks and wants the AI-generated plan and pantry-aware suggestions.

**Steps:**

1. Review the paid tier — Maya sees exactly what upgrading unlocks: the AI weekly plan, pantry-aware suggestions, and learning from ratings — and that everything she already has stays hers if she ever downgrades. [RESEARCH-INFORMED: retroactive paywalling of users' own history produced a documented trust backlash (Cozi, Trustpilot average 2.1/5, HIGH confidence)]
2. Subscribe — She chooses monthly billing and enters payment details.
3. Features unlock — AI plan generation becomes available immediately; her next Sunday brings a generated plan instead of an empty week to fill in manually.
4. Later, review account data — Weeks later, Maya requests an export of the household's data out of curiosity about what Plateful holds, and reviews it.

**Failure/Recovery Variant:** A renewal payment fails a few months in. Maya is not locked out immediately — she receives a clear grace-period notice, updates her expired card, and the subscription continues without any gap in the AI plan.

**Success Outcome:** Maya unlocks the paid features she wants with a clear understanding of cost and value, and can see and control exactly what data the household has accumulated, consistent with the brief's privacy promise.

**Connected Features:** Subscription & Billing Management, Account & Data Management, Manual Weekly Planning

---

### Free-Tier Manual Week

**Owning Persona:** Maya

**Coverage:** Regular

**Journey Goal:** Plan the coming week's dinners by hand on the free tier and have the shared grocery list build itself from the picks. [AUDIT-ADDED: 2 -- journey added for Manual Weekly Planning, the free tier's planning experience named in BRIEF.md, Business Context]

**Entry Point:** It is Sunday afternoon; Maya's household is on the free tier and next week's seven nights are empty.

**Steps:**

1. Open next week — Maya sees seven empty nights, each with a prompt to pick a dinner, and the weekly budget at the top.
2. Pick dinners — She taps Monday, searches the recipe library for "quick chicken", and picks a 20-minute dish; she fills four more nights the same way, watching the week's estimated total against budget.
3. Accept a suggestion — Sam has suggested a pasta bake for Wednesday; Maya accepts it with one tap.
4. Leave a night open — She leaves Saturday empty for a takeaway treat; it shows a gentle "nothing planned" marker, not an error.
5. See the list — The shared grocery list has already filled in with the ingredients for the six planned dinners, grouped by aisle.

**Failure/Recovery Variant:** Maya tries to pick a satay noodle dish she saw in the library for Thursday. It is marked ineligible with a plain reason — "contains peanuts — not safe for Jordan" — and cannot be placed. She taps a suggested nut-free alternative instead and the week carries on without her having to check the ingredients herself.

**Success Outcome:** Maya has a safe, budget-aware week of dinners and a ready-made shared list, without paying and without an unsafe dinner ever reaching the plan.

**Connected Features:** Manual Weekly Planning, Recipe Library (Starter Recipes), Dietary Rules & Allergy Safety Engine, Shared Grocery List, One-Tap Meal Swap

---

### End-of-Week Check-In & Inviting Another Family

**Owning Persona:** Sam

**Coverage:** Regular

**Journey Goal:** Record how the week went on food waste and spend, and share Plateful with a friend's family. [AUDIT-ADDED: 3 -- journey added for Weekly Waste & Spend Check-In and Invite Another Household, both added to close gaps in the brief's success criteria]

**Entry Point:** On Saturday evening Sam opens Plateful and sees the week's check-in card at the top of the plan.

**Steps:**

1. Answer the check-in — Sam taps "a little" for food thrown away this week and enters a rough grocery spend; it takes two taps and a number.
2. See the trend — The card shows the household's last few weeks against its starting point and its budget: less wasted than before Plateful, and spend just under budget.
3. Share with a friend — A friend from the school gate has been complaining about Sunday planning; Sam taps "invite another family" and shares his personal link in their chat.
4. Friend joins — Later, Sam sees a note that a family he invited has set up their own household.

**Failure/Recovery Variant:** Sam's friend opens the link but, busy that evening, only finishes setting up her household four days later on the same phone. The referral still counts because setup finished within 30 days, and Sam gets his note then; nothing needed resending.

**Success Outcome:** The household has a record of how much less it is wasting and spending, and another family has joined through a household invitation — the growth path the brief expects.

**Connected Features:** Weekly Waste & Spend Check-In, Invite Another Household
