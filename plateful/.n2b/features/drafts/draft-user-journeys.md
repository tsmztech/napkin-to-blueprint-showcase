---
document_type: user-journeys
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# User Journeys

## Journey

This document contains 8 journeys covering the household's full lifecycle: first setup, the weekly planning loop, shopping, weeknight dinners, recipe and pantry management, allergy-safe recovery, joining a household, and account/billing management. Journeys are owned by Maya (organiser) and Sam (other adult member); every Core and Important feature appears across the set, and all three coverage types (First-use, Regular, Edge/Recovery) are represented.

---

### First Household Setup

**Owning Persona:** Maya

**Coverage:** First-use

**Journey Goal:** Get the household fully set up — members, dietary rules, budget, and schedule — so the first weekly plan can be generated.

**Entry Point:** Maya creates a Plateful account for the first time, motivated by wanting to stop re-planning dinner from scratch every Sunday.

**Steps:**

1. Create the household — Maya names the household and becomes its first member. She sees a short, guided setup rather than a long form.
2. Add members and rules — Maya adds her partner and two kids, entering each person's allergies, diets, and dislikes, including her child's nut allergy as a hard rule.
3. Set the practical facts — Maya states the weekly food budget and marks which weeknights are short on time.
4. Set units, currency and aisle layout — Maya confirms her local units, currency, and default supermarket aisle groupings.
5. Setup complete — Maya sees a clear confirmation that the household is ready, and that the first plan is on its way.

**Failure/Recovery Variant:** Maya is interrupted partway through entering member dietary rules and closes the app. When she reopens Plateful later, setup resumes exactly where she left off — her partner's and first child's rules are preserved, and she is not asked to re-enter anything, consistent with FEAT-01's partial-setup handling.

**Success Outcome:** Within one sitting, the household exists with accurate allergy and diet data, a budget, a schedule, and locale settings — everything the AI weekly plan needs to generate a safe, realistic first week.

**Connected Features:** Household Setup & Member Profiles, Dietary Rules & Allergy Safety Engine, Household Invitations & Membership, Units, Currency & Locale Configuration

---

### Sunday Plan Review & First Swap

**Owning Persona:** Maya

**Coverage:** Regular

**Journey Goal:** Review the week's AI-generated plan, adjust one meal that doesn't fit, and let the household see it.

**Entry Point:** A Sunday-evening notification arrives: "Next week's plan is ready."

**Steps:**

1. Open the notification — Maya taps it and lands directly on the new week's plan.
2. Scan the week — She sees seven dinners, each with cook time, rough cost, and a "checked against allergies" badge; Thursday calls out that it uses spinach and feta already in the fridge.
3. Check kid input — Because her older child has a limited login, she sees that child's vote reflected in one night's chosen dinner.
4. Swap a meal — Monday's curry looks too slow for a busy night, so she taps swap and picks a faster, already-safety-checked alternative in one tap.
5. Confirm and share — The plan and grocery list update instantly; the rest of the household can now see the finalized week.

**Failure/Recovery Variant:** After Maya swaps Monday's meal, the swap fails to save due to a dropped connection. The original meal remains visible rather than an empty slot, and Maya is offered an immediate retry; the second attempt succeeds and the list updates correctly.

**Success Outcome:** The week's plan reflects the household's real schedule with no unsafe suggestions, and the whole household can see it — all within a few minutes of the notification arriving.

**Connected Features:** AI Weekly Dinner Plan Generation, One-Tap Meal Swap, Weekly Plan Ready Notification, Dietary Rules & Allergy Safety Engine, Older-Kid Dinner Voting

---

### Supermarket Shared Shopping

**Owning Persona:** Sam

**Coverage:** Regular

**Journey Goal:** Do the week's shopping efficiently using the shared, aisle-grouped grocery list, even with a weak in-store signal.

**Entry Point:** Sam arrives at the supermarket on Saturday with the shared grocery list already open on their phone.

**Steps:**

1. Open the list — Sam sees the combined list, grouped by aisle, generated automatically from the week's plan.
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
3. Rate after dinner — After eating, the household members each give a quick thumbs up or down; the kids tap their own ratings.
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
3. See the explanation — A short note explains that options are limited this week given the household's combined rules, rather than the app pretending there were more choices.
4. Pick one — Maya selects one of the two safe options, and the plan and grocery list update immediately.

**Failure/Recovery Variant:** Maya first tries swapping into a recipe she remembers cooking before — but it isn't offered as an option, because its ingredient data can't be fully verified against her child's allergy. Rather than showing it and risking an unsafe suggestion, the app excludes it silently and explains, when she searches for it directly in the library, why it isn't eligible for her household's plan.

**Success Outcome:** Maya successfully swaps Wednesday's dinner, and at no point does an unsafe meal appear as an option — the allergy rule holds even under a highly constrained week.

**Connected Features:** Dietary Rules & Allergy Safety Engine, One-Tap Meal Swap

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
4. Understand the role — A short explanation clarifies what Sam can do (view the plan, swap meals, shop, rate) versus what Maya manages (setup, budget, billing).

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

1. Review the paid tier — Maya sees exactly what upgrading unlocks: the AI weekly plan, pantry-aware suggestions, and learning from ratings.
2. Subscribe — She chooses monthly billing and enters payment details.
3. Features unlock — AI plan generation becomes available immediately; her next Sunday brings a generated plan instead of an empty week to fill in manually.
4. Later, review account data — Weeks later, Maya requests an export of the household's data out of curiosity about what Plateful holds, and reviews it.

**Failure/Recovery Variant:** A renewal payment fails a few months in. Maya is not locked out immediately — she receives a clear grace-period notice, updates her expired card, and the subscription continues without any gap in the AI plan.

**Success Outcome:** Maya unlocks the paid features she wants with a clear understanding of cost and value, and can see and control exactly what data the household has accumulated, consistent with the brief's privacy promise.

**Connected Features:** Subscription & Billing Management, Account & Data Management
