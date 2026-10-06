---
document_type: product-features
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# Product Features

## Summary

This product includes 22 features: 9 Core, 9 Important, 4 Nice-to-Have. By phase: 15 MVP, 4 v1, 3 Later. By type: 12 User-Facing, 7 Platform, 3 Lifecycle. The product manages 12 domain entities. Core features cover household setup, the AI weekly plan, allergy safety, swapping, pantry awareness, and the shared grocery list; Important features cover recipe import, leftovers, learning from ratings, reminders, billing, onboarding, localization, older-kid voting, and account/data management; Nice-to-Have features cover plan history and two later-phase integrations plus operator support access.

## Domain Entity Inventory

### Entity: Household
- **Description:** The shared account for one family — its members, dietary rules, budget, schedule, and subscription tier all hang off it. There is one household per account in v1.
- **Lifecycle:** Created -> Active -> (optionally) Closed/Deleted
- **Created by:** Household Setup & Member Profiles (FEAT-01)
- **Managed by:** Household Setup & Member Profiles (FEAT-01), Subscription & Billing Management (FEAT-14), Account & Data Management (FEAT-18)
- **Referenced by:** Nearly every feature in the product

### Entity: Member Profile
- **Description:** One person in the household — organiser, other adult member, or kid profile — with their own dietary rules, dislikes, and (for adults and older kids) their own login.
- **Lifecycle:** Invited -> Active -> (optionally) Removed
- **Created by:** Household Setup & Member Profiles (FEAT-01), Household Invitations & Membership (FEAT-09)
- **Managed by:** Household Setup & Member Profiles (FEAT-01)
- **Referenced by:** Dietary Rules & Allergy Safety Engine (FEAT-02), AI Weekly Dinner Plan Generation (FEAT-03), Meal Rating & Preference Learning (FEAT-12), Older-Kid Dinner Voting (FEAT-17)

### Entity: Dietary Rule
- **Description:** A hard or soft constraint tied to a Member Profile — an allergy, a religious rule (e.g., halal), a per-person vegetarian setting, or a learned dislike.
- **Lifecycle:** Created -> Active -> (optionally) Edited or Removed
- **Created by:** Household Setup & Member Profiles (FEAT-01)
- **Managed by:** Household Setup & Member Profiles (FEAT-01), Meal Rating & Preference Learning (FEAT-12) (for learned dislikes)
- **Referenced by:** Dietary Rules & Allergy Safety Engine (FEAT-02), AI Weekly Dinner Plan Generation (FEAT-03)

### Entity: Weekly Plan
- **Description:** The set of seven dinners (and their leftover-to-lunch links) proposed for one household for one week.
- **Lifecycle:** Generated -> Reviewed -> Active (through the week) -> Archived
- **Created by:** AI Weekly Dinner Plan Generation (FEAT-03)
- **Managed by:** AI Weekly Dinner Plan Generation (FEAT-03), One-Tap Meal Swap (FEAT-04)
- **Referenced by:** Weekly Plan Ready Notification (FEAT-07), Shared Grocery List (FEAT-06), Leftover Rollover to Lunches (FEAT-11), Tonight's Dinner Reminder (FEAT-13), Older-Kid Dinner Voting (FEAT-17), Weekly Plan History (FEAT-19)

### Entity: Planned Meal
- **Description:** One dinner slot within a Weekly Plan — a specific night, a specific recipe, its safety badges, and its swap history.
- **Lifecycle:** Proposed -> Confirmed -> (optionally) Swapped -> Cooked
- **Created by:** AI Weekly Dinner Plan Generation (FEAT-03)
- **Managed by:** One-Tap Meal Swap (FEAT-04)
- **Referenced by:** Shared Grocery List (FEAT-06), Leftover Rollover to Lunches (FEAT-11), Tonight's Dinner Reminder (FEAT-13), Meal Rating & Preference Learning (FEAT-12)

### Entity: Recipe
- **Description:** A cookable dish — ingredients, steps, cook time, rough cost, and dietary badges — either from the starter library or imported from a web link.
- **Lifecycle:** Added -> Active -> (optionally) Archived
- **Created by:** Recipe Library (Starter Recipes) (FEAT-08) (starter content), Recipe Import from Web Link (FEAT-10) (user-imported)
- **Managed by:** Recipe Library (Starter Recipes) (FEAT-08)
- **Referenced by:** AI Weekly Dinner Plan Generation (FEAT-03), Dietary Rules & Allergy Safety Engine (FEAT-02), Shared Grocery List (FEAT-06)

### Entity: Pantry Item
- **Description:** Something the household says it already has on hand, used to steer plan generation away from buying duplicates.
- **Lifecycle:** Added -> Active -> (optionally) Used/Removed
- **Created by:** Pantry-Aware Suggestions (FEAT-05)
- **Managed by:** Pantry-Aware Suggestions (FEAT-05)
- **Referenced by:** AI Weekly Dinner Plan Generation (FEAT-03), Shared Grocery List (FEAT-06)

### Entity: Grocery List
- **Description:** The one combined, aisle-grouped shopping list generated from the week's plan and shared live by the whole household.
- **Lifecycle:** Generated -> Active (through the week) -> Archived
- **Created by:** Shared Grocery List (FEAT-06)
- **Managed by:** Shared Grocery List (FEAT-06)
- **Referenced by:** Weekly Plan History (FEAT-19), Online Grocery Ordering Handoff (FEAT-20)

### Entity: Grocery List Item
- **Description:** One line on the Grocery List — an ingredient from the plan or something a household member added directly — with its aisle, quantity, and ticked/unticked state.
- **Lifecycle:** Added -> Unticked -> Ticked -> (list archive) Cleared
- **Created by:** Shared Grocery List (FEAT-06) (from the plan or manual add)
- **Managed by:** Shared Grocery List (FEAT-06)
- **Referenced by:** Online Grocery Ordering Handoff (FEAT-20)

### Entity: Rating
- **Description:** A household member's thumbs up/down on a cooked meal, used to learn what the family actually likes.
- **Lifecycle:** Created -> Active (feeds future plans)
- **Created by:** Meal Rating & Preference Learning (FEAT-12)
- **Managed by:** Meal Rating & Preference Learning (FEAT-12)
- **Referenced by:** AI Weekly Dinner Plan Generation (FEAT-03)

### Entity: Invitation
- **Description:** An outstanding invite from the organiser to another adult (or, later, an older kid) to join the household.
- **Lifecycle:** Sent -> Accepted or Expired
- **Created by:** Household Invitations & Membership (FEAT-09)
- **Managed by:** Household Invitations & Membership (FEAT-09)
- **Referenced by:** Member Onboarding (FEAT-15)

### Entity: Subscription
- **Description:** The household's plan tier (free or paid, monthly or yearly) and its billing state.
- **Lifecycle:** Free -> (optionally) Upgraded -> Active (paid) -> (optionally) Downgraded/Cancelled
- **Created by:** Household Setup & Member Profiles (FEAT-01) (defaults to free)
- **Managed by:** Subscription & Billing Management (FEAT-14)
- **Referenced by:** AI Weekly Dinner Plan Generation (FEAT-03), Pantry-Aware Suggestions (FEAT-05), Meal Rating & Preference Learning (FEAT-12)

## Core Features

### Household Setup & Member Profiles

**ID:** FEAT-01

**Description:** The organiser sets up the household once: who eats with them, each person's allergies and diet, the weekly food budget, and how much time there is on which nights. This is the foundation every other feature reads from.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief states the household "is set up once" before anything else can happen (BRIEF.md, Vision). Without this, there is no household to plan for. MVP: nothing else in the product functions without it.

**Connected Entities:** Household (create, update), Member Profile (create, update), Dietary Rule (create, update)

**Key Capabilities:**
- Create the household — Organiser names the household and becomes its first member
- Add member profiles — Organiser adds each person eating with them, including kid profiles with no login
- Set dietary rules per person — Organiser records allergies, religious rules, per-person vegetarian settings, and known dislikes
- Set the weekly budget — Organiser states a rough weekly food budget
- Set the weekly schedule — Organiser marks which nights are short on time (e.g., 30-minute weeknights)
- Edit setup later — Organiser revisits and changes any of the above at any time

**Primary Flows & Alternates:**
- Happy path: organiser opens setup -> names the household -> adds members and their dietary rules -> sets budget and schedule -> setup complete, ready for the first plan
- Partial setup: organiser can save and return later; incomplete setup blocks plan generation only for the specific facts genuinely missing (e.g., no schedule set defaults to no time constraint, not a hard block)
- Later edit: organiser adds a new member, changes an allergy, or adjusts the budget after the household is already active — changes apply to the next generated plan, not retroactively

**States:** Empty: a brand-new household shows a short, guided setup rather than a blank form. Loading: saving a setup step shows an inline confirmation, not a full-page spinner. Error: a failed save keeps the entered data on screen and offers a retry, never silently discards input. Offline-degraded: setup can be drafted offline and is held locally until connectivity returns to save.

**Validation & Limits:** Household name required (1–60 characters); at least one member (the organiser) required; each member's dietary rules are optional but an allergy, once entered, cannot be silently dropped without an explicit confirmation step; weekly budget must be a positive amount in the household's configured currency.

**Access:** Maya (Organiser) has Full access per the Access Matrix in user-persona.md. Sam (Other Adult Member) has View-only access — they can see but not change household facts. Jordan (Kid Profile) has no access. An unauthorized visitor sees only an invitation-acceptance screen, never household data.

**Communications:** A one-time confirmation when setup is completed enough to generate a first plan.

**Data Notes:** Captured: household name, member list, per-member dietary rules, weekly budget, weekly schedule. Displayed: the current setup state to the organiser. Derived: none. Source: organiser input only.

**Interactions:** Feeds every other feature; Household Invitations & Membership (FEAT-09) extends the member list it creates; Dietary Rules & Allergy Safety Engine (FEAT-02) reads the dietary rules captured here.

**Signals:** household_created, member_added, dietary_rule_added, budget_set, schedule_set, setup_completed.

### Dietary Rules & Allergy Safety Engine

**ID:** FEAT-02

**Description:** Before any suggested meal reaches a household member, the app itself checks it against every household member's allergies and religious rules. This check runs independently of the AI — the AI proposes, the app verifies — so a single AI mistake can never reach the table.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief states this as a hard rule, not a preference: "the app itself checks every suggestion against the household's allergy list... the AI is never the last line of defence" (BRIEF.md, Vision, Constraints: Safety). This is the single most trust-critical feature in the product and must exist from day one.

**Connected Entities:** Dietary Rule (read), Recipe (read), Planned Meal (update — attaches a "checked" badge)

**Key Capabilities:**
- Verify every suggestion — Every recipe considered for a household's plan is checked against that household's allergies and religious rules before it can appear
- Show a safety badge — Every meal in the plan carries a "checked against allergies" badge with a standard "always check labels" disclaimer
- Block unsafe suggestions — A recipe that fails the check for any household member is removed from consideration for that household entirely, not just flagged
- Distinguish rule strength — Allergies and religious rules are hard filters; vegetarian settings apply per person with a shared-meal vegetarian option; dislikes are soft and never block a suggestion

**Primary Flows & Alternates:**
- Happy path: a candidate recipe is checked against all household members' hard rules before it is ever shown; a safe recipe displays its "checked against allergies" badge
- Hard-rule failure: a recipe that would violate any member's allergy or religious rule is silently excluded from that household's candidate pool — never shown, never suggested, never requiring the user to reject it themselves
- Vegetarian handling: a shared meal that isn't inherently vegetarian can carry a "vegetarian option" variant so one household with mixed diets can still share one plan

**States:** Empty: N/A — this feature has no user-facing empty state; it runs invisibly behind every suggestion. Loading: the safety check completes before a suggestion is ever shown, so no separate loading state is visible to the user. Error: if a safety check cannot be completed for a candidate recipe, that recipe is excluded by default rather than shown unchecked — failing closed, never open. Offline-degraded: N/A — the safety check runs as part of plan generation, which requires connectivity; there is no offline mode for generating new suggestions.

**Validation & Limits:** Every household member's allergy and religious-rule data must be checked against every ingredient of every candidate recipe with no partial matching shortcuts; a recipe missing complete ingredient data cannot pass the check and is excluded rather than assumed safe.

**Access:** All household roles (Maya, Sam, Jordan) can see the resulting safety badges per the Access Matrix; only Maya can edit the underlying dietary rules this engine reads. An unauthorized visitor sees nothing, since this engine has no standalone screen.

**Communications:** N/A — this feature runs silently within plan generation and swaps; it sends no notifications of its own, though its badges are visible wherever a meal is shown.

**Data Notes:** Displayed: the "checked against allergies" badge and disclaimer on every meal. Derived: the pass/fail safety determination for each candidate recipe against each household's Dietary Rules. Source: Dietary Rule data from Household Setup (FEAT-01) and ingredient data from each Recipe.

**Interactions:** Runs within AI Weekly Dinner Plan Generation (FEAT-03) and One-Tap Meal Swap (FEAT-04); reads data from Household Setup & Member Profiles (FEAT-01); its badge is displayed wherever Recipe Library (FEAT-08) or Recipe Import (FEAT-10) content is shown.

**Signals:** safety_check_run, safety_check_failed (recipe excluded), safety_badge_shown, safety_check_data_incomplete.

### AI Weekly Dinner Plan Generation

**ID:** FEAT-03

**Description:** Every week, the household receives a proposed 7-day dinner plan that fits everyone's allergies, diets, dislikes, schedule, and budget, and makes use of what the household says is already in the fridge.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** This is the product's central promise: "AI proposes a realistic 7-day dinner plan every week that respects all of it" (BRIEF.md, Vision). It is the paid tier's primary value (BRIEF.md, Business Context) and the feature every other Core feature exists to support.

**Connected Entities:** Weekly Plan (create), Planned Meal (create), Recipe (read), Dietary Rule (read), Pantry Item (read), Rating (read), Subscription (read — gates access)

**Key Capabilities:**
- Generate the week's plan — Household receives seven dinners that respect every hard dietary rule, the stated schedule, and the budget
- Use up the pantry — The plan favors recipes that use ingredients the household has already logged as on hand
- Show cost and time per meal — Each dinner shows its rough cost and cook time
- Learn from ratings over time — Later plans favor meals the household has rated highly and avoid ones rated poorly

**Primary Flows & Alternates:**
- Happy path: plan generates automatically once a week -> household opens it to see seven dinners, each with cook time, rough cost, and dietary badges, and at least one flagged as using existing pantry items
- Insufficient data: a brand-new household with minimal setup (e.g., no pantry items logged) still receives a complete, safe plan — pantry-awareness is a refinement, not a precondition for getting a plan
- Free tier: households on the free tier do not receive an AI-generated plan and instead see an invitation to build a plan manually or upgrade

**States:** Empty: a household that has not yet had a plan generated sees a clear "your first plan is on its way" message rather than a blank week. Loading: plan generation shows a short, explained wait (typically well under a minute) rather than an indefinite spinner. Error: if generation fails, the household keeps the previous week's plan visible and is offered a retry, never left with no plan at all. Offline-degraded: an already-generated plan remains fully viewable offline; generating a new plan requires connectivity.

**Validation & Limits:** Generation requires at least one household member with complete dietary-rule data (even "no restrictions" is an explicit statement) and a schedule; a plan always proposes exactly seven dinners, one per day, regardless of household size.

**Access:** Maya (Organiser) and Sam (Other Adult Member) have Full and View access respectively per the Access Matrix; Jordan (Kid Profile) has View access. An unauthorized visitor sees no plan content.

**Communications:** Triggers Weekly Plan Ready Notification (FEAT-07) once generation completes.

**Data Notes:** Captured: none directly (generation is triggered automatically on a schedule). Displayed: seven Planned Meals with cook time, rough cost, and safety/dietary badges. Derived: the meal selection itself, computed from Recipe data filtered by Dietary Rules & Allergy Safety Engine (FEAT-02), weighted by Pantry Items and past Ratings, and fit to the household's budget and schedule. Source: household setup data, recipe data, pantry data, and rating history.

**Interactions:** Depends on Household Setup & Member Profiles (FEAT-01) for constraints, Dietary Rules & Allergy Safety Engine (FEAT-02) for safety filtering, Pantry-Aware Suggestions (FEAT-05) for pantry weighting, Meal Rating & Preference Learning (FEAT-12) for taste learning, Recipe Library (FEAT-08) and Recipe Import (FEAT-10) for candidate recipes, and Subscription & Billing Management (FEAT-14) for tier gating. Feeds Weekly Plan Ready Notification (FEAT-07), Shared Grocery List (FEAT-06), One-Tap Meal Swap (FEAT-04), Leftover Rollover to Lunches (FEAT-11), and Older-Kid Dinner Voting (FEAT-17).

**Signals:** plan_generation_started, plan_generation_completed, plan_generation_failed, plan_viewed, plan_used_pantry_item (count).

### One-Tap Meal Swap

**ID:** FEAT-04

**Description:** Any dinner in the plan can be replaced with a different one in a single tap, and the grocery list updates immediately to match.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief calls this out explicitly: "Any meal can be swapped with one tap, and the grocery list updates instantly" (BRIEF.md, Vision, The Experience). Plans that cannot flex to real life are abandoned; this keeps the plan usable when reality changes.

**Connected Entities:** Planned Meal (update), Weekly Plan (update), Grocery List (update — indirectly, via FEAT-06)

**Key Capabilities:**
- Swap a meal — Household member replaces one night's dinner with an alternative in one tap
- See safe alternatives only — Every offered alternative has already passed the allergy safety check
- Instant list update — The grocery list adjusts automatically to reflect the swap

**Primary Flows & Alternates:**
- Happy path: user taps swap on a planned meal -> sees a short list of safe alternatives -> picks one -> the plan and grocery list update immediately
- No good alternative available: if the safety and schedule constraints leave very few options, the app still shows what qualifies rather than an empty list, and explains briefly why choices are limited
- Repeated swap: a meal can be swapped more than once in the same week without restriction

**States:** Empty: N/A — a meal to swap always exists once a plan is generated. Loading: alternatives appear within a couple of seconds with a brief inline indicator. Error: a failed swap leaves the original meal in place rather than an empty slot, with a retry option. Offline-degraded: swapping requires connectivity to fetch safe alternatives; while offline, the current plan remains viewable but swap is disabled with a clear explanation.

**Validation & Limits:** An alternative must pass the same allergy/religious hard-rule check as original generation; no more than one active swap operation per meal slot at a time (prevents duplicate swaps from a double tap).

**Access:** Maya (Organiser) and Sam (Other Adult Member) both have Full access per the Access Matrix; Jordan (Kid Profile) has no access to swap, only to view the result. An unauthorized visitor cannot swap anything.

**Communications:** N/A — swapping is an immediate, in-app action with no separate notification.

**Data Notes:** Displayed: the current meal and its safe alternatives. Derived: the alternatives list, computed the same way as original plan generation but scoped to one slot. Source: Recipe data filtered by Dietary Rules & Allergy Safety Engine (FEAT-02).

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) for the plan it modifies and Dietary Rules & Allergy Safety Engine (FEAT-02) for safe alternatives; updates feed Shared Grocery List (FEAT-06) automatically.

**Signals:** meal_swap_opened, meal_swap_completed, meal_swap_failed, meal_swap_alternatives_limited.

### Pantry-Aware Suggestions

**ID:** FEAT-05

**Description:** The household tells Plateful what it already has on hand, and the weekly plan favors recipes that use those ingredients up before they go to waste.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief's central "nothing gets thrown away" promise depends on this (BRIEF.md, Vision, The Experience: "Thursday says 'uses the spinach and feta you already have'"). The brief leaves pantry depth as an open question and leans toward a simple "tell me what I have" model rather than detailed inventory tracking.

**Connected Entities:** Pantry Item (create, update, delete)

**Key Capabilities:**
- Log what's on hand — Household member notes an ingredient they already have, in plain language
- See it used in the plan — A plan that uses a logged pantry item calls that out explicitly (e.g., "uses the spinach and feta you already have")
- Clear used items — Household can mark a pantry item as used up once the week's plan consumes it

**Primary Flows & Alternates:**
- Happy path: household member adds "spinach, feta" before the week's plan generates -> plan uses them in a dinner and calls it out
- Nothing logged: a household that logs no pantry items still receives a complete plan; pantry-awareness simply has nothing to weight toward that week
- Stale item: a logged item that goes unused for several weeks is not automatically removed — the household clears it manually when it is used or gone

**States:** Empty: an empty pantry list shows a short prompt to add what's on hand, not a bare screen. Loading: adding an item confirms instantly. Error: a failed save keeps the typed item visible with a retry option. Offline-degraded: pantry items can be added offline and sync once connectivity returns.

**Validation & Limits:** Item name required (1–80 characters, free text — no rigid inventory schema); no fixed limit on how many items a household can log, though the feature is designed for a short, current list rather than a full inventory.

**Access:** Maya (Organiser) and Sam (Other Adult Member) have Full access per the Access Matrix; Jordan (Kid Profile) has no access. An unauthorized visitor sees no pantry data.

**Communications:** N/A — this feature has no notifications of its own; its effect surfaces as a callout inside the weekly plan.

**Data Notes:** Captured: free-text pantry item names, added by household members. Displayed: the current pantry list and, within the plan, which dinners use a logged item. Derived: none. Source: household input only.

**Interactions:** Feeds AI Weekly Dinner Plan Generation (FEAT-03), which reads current Pantry Items when selecting meals.

**Signals:** pantry_item_added, pantry_item_cleared, pantry_item_used_in_plan.

### Shared Grocery List

**ID:** FEAT-06

**Description:** One combined grocery list, generated from the week's plan and grouped by supermarket aisle, that the whole household sees and ticks off together in real time — including in the store with a weak signal.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief describes this as one of the two concrete deliverables of the product (alongside the plan itself) and states it "must feel instant and must keep working in a supermarket with bad signal" (BRIEF.md, Vision, Scale & Non-Functional Expectations). It directly replaces the group-chat list called out in the Problem Statement.

**Connected Entities:** Grocery List (create, update), Grocery List Item (create, update, delete)

**Key Capabilities:**
- Auto-generate from the plan — The list is built from the ingredients of the week's plan, grouped by aisle
- Add manually — Any household member can add an item the plan didn't include
- Tick off live — Ticking an item is visible to every household member immediately
- Work offline in the store — Changes made with no signal sync automatically once connectivity returns

**Primary Flows & Alternates:**
- Happy path: plan generates -> list builds automatically, grouped by aisle -> household member adds "more yoghurt" from home -> partner ticks items off in the store, seeing the same live-updating list
- Offline edit: a household member ticks off items with no signal; the ticks are held locally and sync the moment connectivity returns, without creating duplicates or conflicts
- Swap-driven update: when a meal is swapped (FEAT-04), the list updates its ingredients automatically without the user having to re-check anything

**States:** Empty: before a plan exists, the list shows a short explanation that it will populate once the first plan is generated, plus the option to add items manually in the meantime. Loading: newly generated list items appear within a couple of seconds with an inline indicator. Error: a failed tick or add retries automatically in the background and only surfaces an error if it cannot eventually succeed. Offline-degraded: full read and tick/add functionality remains available offline; changes queue and sync when connectivity returns.

**Validation & Limits:** Manually added item name required (1–80 characters); aisle grouping uses the household's configured aisle names; duplicate manual entries for the same ingredient are merged rather than shown twice.

**Access:** Maya (Organiser) and Sam (Other Adult Member) have Full access; Jordan (Kid Profile) has Full access to add and tick items but not to manage household-level list settings (aisle names, which is part of FEAT-01/FEAT-16). An unauthorized visitor cannot see the list.

**Communications:** N/A — updates are shown live in the list itself rather than pushed as separate notifications.

**Data Notes:** Captured: manually added items and tick state. Displayed: the combined, aisle-grouped list. Derived: the auto-generated portion of the list, computed from the week's Planned Meals' ingredients minus logged Pantry Items. Source: AI Weekly Dinner Plan Generation (FEAT-03) output, Pantry-Aware Suggestions (FEAT-05) data, and direct household input.

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) and One-Tap Meal Swap (FEAT-04) for its auto-generated contents and Pantry-Aware Suggestions (FEAT-05) to avoid listing items already on hand; read by Weekly Plan History (FEAT-19) and, in a later phase, Online Grocery Ordering Handoff (FEAT-20).

**Signals:** grocery_list_generated, grocery_item_added_manually, grocery_item_ticked, grocery_item_synced_after_offline.

### Weekly Plan Ready Notification

**ID:** FEAT-07

**Description:** The household is told, on a predictable schedule, that next week's plan is ready to review.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief's own description of the core experience begins here: "It's Sunday evening and a notification arrives: 'Next week's plan is ready.'" (BRIEF.md, The Experience). Without this, the weekly plan is invisible until someone happens to check.

**Connected Entities:** Weekly Plan (read)

**Key Capabilities:**
- Notify when the plan is ready — Household members with notifications enabled are told as soon as generation completes
- Control who gets notified — Each household member can enable or disable this notification for themselves

**Primary Flows & Alternates:**
- Happy path: plan generation completes -> enabled household members receive a "next week's plan is ready" notification -> tapping it opens the plan directly
- Notification disabled: a household member who has turned this off simply sees the plan the next time they open the app, with no notification sent
- Delayed delivery: if notification delivery itself fails or is delayed, the plan is still available in-app immediately upon generation, unaffected by the notification path

**States:** Empty: N/A — this feature has no content of its own beyond the notification text. Loading: N/A — delivery is near-instant once generation completes. Error: a failed notification delivery does not block or delay the plan's in-app availability. Offline-degraded: a notification sent while a device is offline is delivered once the device reconnects, per standard device-notification behavior.

**Validation & Limits:** At most one "plan ready" notification per household per week, tied to generation completion, not user action.

**Access:** Each household member (Maya, Sam) controls only their own notification preference per the Access Matrix; Jordan (Kid Profile) has no notification settings since kid profiles have no login by default. An unauthorized visitor receives nothing.

**Communications:** This feature is itself the communication: a device notification, "Next week's plan is ready."

**Data Notes:** Displayed: N/A — this is a notification, not a data view. Derived: none. Source: the completion event from AI Weekly Dinner Plan Generation (FEAT-03).

**Interactions:** Triggered by AI Weekly Dinner Plan Generation (FEAT-03); its preference toggle lives within Household Setup & Member Profiles (FEAT-01).

**Signals:** plan_ready_notification_sent, plan_ready_notification_opened, plan_ready_notification_disabled.

### Recipe Library (Starter Recipes)

**ID:** FEAT-08

**Description:** A built-in library of starter recipes the AI plan draws from and that household members can browse directly.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief names "a starter recipe library" explicitly as part of the recipe sources the product needs (BRIEF.md, Ecosystem & Integrations). Without a starter library, plan generation would have nothing to propose from day one, before any household has imported recipes of their own.

**Connected Entities:** Recipe (create — starter content, read)

**Key Capabilities:**
- Browse recipes — Household member looks through the library directly, outside the weekly plan
- View recipe detail — Household member sees ingredients, steps, cook time, rough cost, and dietary badges for one recipe
- Search and filter — Household member finds recipes by name, ingredient, or dietary badge

**Primary Flows & Alternates:**
- Happy path: household member opens the library -> searches or filters -> opens a recipe's detail view with ingredients, steps, and badges
- No results: a search with no matches shows a plain "nothing found" message and suggests broadening the search, never an empty white screen
- Cross-reference from plan: opening a recipe from within the weekly plan shows the same detail view as browsing the library directly

**States:** Empty: N/A — the starter library ships pre-populated and is never empty for any household. Loading: search results appear within about a second. Error: a failed detail-view load offers a retry without losing the search context. Offline-degraded: previously viewed recipes remain available offline; new searches require connectivity.

**Validation & Limits:** Search query has no minimum length; results are capped to a manageable page size with further results loaded on scroll.

**Access:** Maya, Sam, and Jordan all have at least View access per the Access Matrix (Maya and Sam Full, Jordan View); an unauthorized visitor cannot browse the library.

**Communications:** N/A — browsing is a self-initiated, in-app action with no notifications.

**Data Notes:** Displayed: recipe name, ingredients, steps, cook time, rough cost, dietary badges. Derived: dietary badges, computed by Dietary Rules & Allergy Safety Engine (FEAT-02) against the household's rules when viewed. Source: starter content plus recipes added via Recipe Import from Web Link (FEAT-10).

**Interactions:** Feeds AI Weekly Dinner Plan Generation (FEAT-03) as its candidate pool alongside Recipe Import from Web Link (FEAT-10); its badges depend on Dietary Rules & Allergy Safety Engine (FEAT-02).

**Signals:** recipe_library_opened, recipe_searched, recipe_detail_viewed, recipe_search_empty.

### Household Invitations & Membership

**ID:** FEAT-09

**Description:** The organiser invites other adults to join the household, so everyone sees the same plan and the same grocery list.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief states plainly that "everyone in a household sees the same plan and grocery list" (BRIEF.md, Target Users & Roles) and that growth is expected to come from "households inviting other households" (BRIEF.md, Business Context) — membership within a household is the mechanism that makes shared use possible at all.

**Connected Entities:** Invitation (create, update), Member Profile (create — on acceptance)

**Key Capabilities:**
- Send an invitation — Organiser invites another adult by a simple, shareable method
- Accept an invitation — Invited adult joins the household and gains their role's access
- Manage outstanding invitations — Organiser can see and revoke a pending invitation

**Primary Flows & Alternates:**
- Happy path: organiser sends an invitation -> invited adult accepts -> a new Member Profile is created for them with Other Adult Member access
- Expired invitation: an invitation not accepted within a reasonable window expires automatically and can be resent
- Revoked invitation: organiser revokes a pending invitation before it is accepted; the invited person sees a clear "this invitation is no longer valid" message if they try to use it afterward

**States:** Empty: a household with no outstanding invitations shows a plain "invite someone" prompt rather than an empty list with no explanation. Loading: sending an invitation confirms within a couple of seconds. Error: a failed invitation send preserves the entered contact detail and offers a retry. Offline-degraded: composing an invitation requires connectivity to send; a drafted invitation is held locally until it can be sent.

**Validation & Limits:** A contact detail is required to send an invitation; a household may have multiple outstanding invitations at once; an already-active member cannot be re-invited.

**Access:** Only Maya (Organiser) can send or revoke invitations per the Access Matrix (household setup access is Full for Maya, View for Sam, None for Jordan); an accepted invitee gains Other Adult Member access automatically. An unauthorized visitor can only act on a specific invitation link addressed to them.

**Communications:** Sends the invitation itself (e.g., a shareable link or message) and a confirmation to the organiser once it is accepted.

**Data Notes:** Captured: the invited person's contact detail and the invitation's state. Displayed: outstanding and accepted invitations to the organiser. Derived: none. Source: organiser input.

**Interactions:** Extends the Member Profile list created by Household Setup & Member Profiles (FEAT-01); feeds Member Onboarding (FEAT-15) for the accepted invitee's first-use experience.

**Signals:** invitation_sent, invitation_accepted, invitation_revoked, invitation_expired.

## Important Features

### Recipe Import from Web Link

**ID:** FEAT-10

**Description:** A household member can save a recipe from any website by pasting its link, adding it to their own recipe pool alongside the starter library.

**Priority:** Important

**Phase:** v1

**Type:** User-Facing

**Rationale:** The brief names this directly: "saving recipes from any website by pasting a link" (BRIEF.md, Ecosystem & Integrations), while also flagging that "whether importing recipes from other sites is legal is an open question" (BRIEF.md, Open Questions). Phased to v1 rather than MVP because the starter library (FEAT-08) already gives new households a working candidate pool on day one; import extends personalization once the core loop is proven and the legal question is resolved.

**Connected Entities:** Recipe (create — imported)

**Key Capabilities:**
- Import by link — Household member pastes a web link and the recipe's ingredients, steps, and cook time are extracted into the household's recipe pool
- Review before saving — Household member confirms or edits extracted details before the recipe is saved
- See imported recipes alongside starter ones — Imported recipes appear in the same library and are eligible for the weekly plan

**Primary Flows & Alternates:**
- Happy path: household member pastes a link -> the recipe's details are extracted -> member reviews and confirms -> it is saved to the household's recipe pool and can appear in future plans
- Extraction failure: a link that cannot be parsed prompts the member to enter the recipe details manually instead of failing silently
- Duplicate import: importing a link already saved for the household surfaces the existing recipe rather than creating a duplicate

**States:** Empty: N/A — this feature has no standing list of its own; imported recipes appear inside Recipe Library (FEAT-08). Loading: extraction shows a brief, explained progress indicator (typically a few seconds). Error: a failed extraction offers manual entry as a fallback rather than a dead end. Offline-degraded: importing requires connectivity to fetch and parse the link; a pasted link is held as a draft if offline and processed once reconnected.

**Validation & Limits:** A well-formed web address is required; extracted ingredient and step text is capped to a reasonable length consistent with other recipes in the library.

**Access:** Maya (Organiser) and Sam (Other Adult Member) have Full access to import per the Access Matrix's Recipe Library row; Jordan (Kid Profile) has View-only access and cannot import. An unauthorized visitor cannot import.

**Communications:** N/A — import is a self-initiated, in-app action with no notifications.

**Data Notes:** Captured: the source link and the member's confirmed/edited recipe details. Displayed: the imported recipe within the library. Derived: the initial extraction (ingredients, steps, cook time) from the linked page, subject to member review before saving. Source: the external web page plus member confirmation.

**Interactions:** Feeds Recipe Library (FEAT-08) and, through it, AI Weekly Dinner Plan Generation (FEAT-03); its badges depend on Dietary Rules & Allergy Safety Engine (FEAT-02) once saved.

**Signals:** recipe_import_started, recipe_import_succeeded, recipe_import_failed, recipe_import_manual_fallback.

### Leftover Rollover to Lunches

**ID:** FEAT-11

**Description:** Leftovers from a planned dinner are carried forward as a suggested lunch on a following day, so extra portions get eaten instead of thrown away.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief names this directly as part of the core vision: "Leftovers roll into lunches" (BRIEF.md, Vision), and the "nothing gets thrown away" moment in the Experience section depends on it. Important rather than Core because the plan and grocery list function completely without it, but it is included at MVP because it is a defining part of the brief's stated experience, not a later refinement.

**Connected Entities:** Planned Meal (read, create — the lunch suggestion)

**Key Capabilities:**
- Suggest a leftover lunch — A dinner cooked with extra portions is offered as a lunch suggestion on a following day
- Confirm or skip — Household member confirms the leftover lunch happened or skips it if it did not

**Primary Flows & Alternates:**
- Happy path: a dinner is planned with visible leftover potential -> the plan suggests it as lunch the next day -> household member confirms it happened
- Skipped leftovers: household member marks the leftover lunch as skipped (e.g., the food was finished at dinner) with no penalty or follow-up prompt
- No leftovers planned: a dinner not expected to produce leftovers simply has no lunch suggestion attached — this is not an error state

**States:** Empty: a day with no leftover lunch suggested shows nothing extra — this is the normal case, not an empty state needing explanation. Loading: N/A — the suggestion is generated as part of plan generation, with no separate loading step. Error: N/A — this is a lightweight suggestion; a failure to compute it simply omits the suggestion rather than showing an error. Offline-degraded: an already-generated leftover suggestion remains viewable offline.

**Validation & Limits:** A leftover suggestion links to exactly one source Planned Meal and one following day; it cannot be scheduled more than two days after the source dinner.

**Access:** Maya and Sam have Full access to confirm/skip per the Access Matrix's Weekly Plan row; Jordan (Kid Profile) has View-only access. An unauthorized visitor cannot see it.

**Communications:** N/A — the suggestion appears within the plan itself; no separate notification is sent.

**Data Notes:** Displayed: the leftover lunch suggestion attached to a day. Derived: which dinners produce leftover-worthy portions and which following day to suggest, computed from Planned Meal data. Source: AI Weekly Dinner Plan Generation (FEAT-03) output.

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) for the dinners it extends; affected by One-Tap Meal Swap (FEAT-04) when a source dinner is swapped.

**Signals:** leftover_lunch_suggested, leftover_lunch_confirmed, leftover_lunch_skipped.

### Meal Rating & Preference Learning

**ID:** FEAT-12

**Description:** Household members rate meals with a simple thumbs up or down after dinner, and future plans learn from those ratings to better match what the family actually likes.

**Priority:** Important

**Phase:** v1

**Type:** User-Facing

**Rationale:** The brief states the plan "learns what the family actually likes" over time and depicts kids giving "a thumbs up or down" after dinner (BRIEF.md, Vision, The Experience). Phased to v1 because meaningful learning requires a history of ratings that does not exist in a household's first weeks; MVP plan generation must work well from ratings alone being absent.

**Connected Entities:** Rating (create), Dietary Rule (update — for learned dislikes)

**Key Capabilities:**
- Rate a meal — Household member gives a thumbs up or down after a dinner is cooked
- See ratings reflected in future plans — Meals rated poorly by the household appear less often; well-liked meals appear more often
- Rate individually — Each household member's rating is recorded separately, so the plan can learn different preferences per person

**Primary Flows & Alternates:**
- Happy path: after dinner, household member gives a thumbs up or down -> the rating is recorded against that meal and that member -> future plan generation weights accordingly
- No rating given: a meal that goes unrated is simply treated as neutral — it is never assumed liked or disliked, and the household is never pressed to rate
- Repeated dislike: a meal rated down repeatedly by the same member is treated increasingly like a soft dislike, feeding back into that member's Dietary Rule data

**States:** Empty: a meal with no ratings yet shows the plain rating prompt with no history. Loading: submitting a rating confirms instantly. Error: a failed rating submission retries automatically in the background. Offline-degraded: a rating given offline is held locally and synced once connectivity returns.

**Validation & Limits:** One rating per household member per Planned Meal; a rating can be changed after submission up until the plan is archived.

**Access:** Maya, Sam, and Jordan (as an older kid, per the brief's "older kids want to vote... rate meals") each rate on an Own-only basis per the Access Matrix; no one sees another member's individual rating broken out, only the aggregate effect on future plans. An unauthorized visitor cannot rate.

**Communications:** N/A — rating is a self-initiated, in-app action with no notifications of its own.

**Data Notes:** Captured: a per-member, per-meal thumbs up/down. Displayed: N/A to other members individually (aggregate effect only, via future plans). Derived: the preference weighting applied during AI Weekly Dinner Plan Generation (FEAT-03), and any resulting soft-dislike update to Dietary Rule data. Source: household member input.

**Interactions:** Feeds AI Weekly Dinner Plan Generation (FEAT-03) directly and updates Dietary Rule data managed by Household Setup & Member Profiles (FEAT-01).

**Signals:** meal_rated_up, meal_rated_down, rating_changed, learned_dislike_applied.

### Tonight's Dinner Reminder

**ID:** FEAT-13

**Description:** On the day of a planned dinner, the household gets a brief nudge with what's cooking and any prep reminder it needs (e.g., taking something out of the freezer).

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief depicts this directly: "On Wednesday at 5pm a nudge arrives: 'Tonight: 20-minute pasta — take the chicken out of the freezer.'" (BRIEF.md, The Experience). Included at MVP because it is core to closing the "what's for dinner?" problem the brief opens with, not a later refinement.

**Connected Entities:** Planned Meal (read)

**Key Capabilities:**
- Send the day's nudge — Household members with notifications enabled are told what's for dinner at a sensible time before cooking
- Include a prep reminder — If a dinner needs early prep (e.g., defrosting), the nudge calls it out
- Control who gets nudged — Each household member can enable or disable this nudge for themselves

**Primary Flows & Alternates:**
- Happy path: nudge fires at a sensible pre-dinner time -> household member sees what's cooking and any prep step needed
- No prep needed: a dinner with no early-prep requirement sends a nudge naming the meal only, with no invented prep step
- Meal swapped same-day: if the dinner is swapped after the nudge was sent, a brief follow-up correction is sent rather than leaving a stale nudge uncorrected

**States:** Empty: N/A — a nudge only fires when a dinner exists for that day. Loading: N/A — delivery is near-instant. Error: a failed nudge delivery does not block the meal from being visible in-app. Offline-degraded: a nudge sent while offline is delivered once the device reconnects, per standard device-notification behavior.

**Validation & Limits:** At most one nudge per household per day, tied to that day's Planned Meal; a same-day swap triggers at most one follow-up correction.

**Access:** Each household member (Maya, Sam) controls their own nudge preference; Jordan (Kid Profile) has no notification settings, consistent with having no login. An unauthorized visitor receives nothing.

**Communications:** This feature is itself the communication: a device notification naming the day's dinner and any prep step.

**Data Notes:** Displayed: N/A — this is a notification, not a data view. Derived: the prep-reminder text, computed from the Planned Meal's recipe requirements (e.g., a frozen ingredient). Source: AI Weekly Dinner Plan Generation (FEAT-03) and One-Tap Meal Swap (FEAT-04) output.

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) and One-Tap Meal Swap (FEAT-04) for the meal it announces; its preference toggle lives within Household Setup & Member Profiles (FEAT-01).

**Signals:** dinner_nudge_sent, dinner_nudge_opened, dinner_nudge_correction_sent, dinner_nudge_disabled.

### Subscription & Billing Management

**ID:** FEAT-14

**Description:** The household can see its current plan tier, upgrade to the paid subscription to unlock the AI weekly plan and pantry-aware suggestions, and manage billing.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** The brief states the business model directly: a free tier for manual planning and the shared list, and "a paid household subscription (monthly or yearly)" that adds the AI plan, pantry suggestions, and learning (BRIEF.md, Business Context). Included at MVP because the product cannot generate any revenue, and the founder needs "paying households within about three months" (BRIEF.md, Constraints: Team/timeline), without it.

**Connected Entities:** Subscription (create, update)

**Key Capabilities:**
- View current tier — Household sees whether it is on the free or paid tier and what each includes
- Upgrade to paid — Organiser subscribes monthly or yearly to unlock AI features
- Manage billing — Organiser updates payment details and views billing history
- Downgrade or cancel — Organiser can move back to the free tier, with a clear explanation of what is lost

**Primary Flows & Alternates:**
- Happy path: organiser reviews what the paid tier unlocks -> chooses monthly or yearly -> subscribes -> AI plan generation and pantry-aware suggestions unlock immediately
- Payment failure: a failed renewal payment does not immediately cut off access — the household is given a clear grace-period notice and a chance to update payment details before losing paid features
- Downgrade: organiser downgrades to free; the household keeps its existing plan and list read-only but stops receiving new AI-generated plans going forward

**States:** Empty: N/A — every household has a tier from creation (defaulting to free). Loading: an upgrade action confirms within a few seconds. Error: a failed upgrade attempt preserves the chosen plan option and offers a retry rather than losing the selection. Offline-degraded: viewing current tier works offline; upgrading or changing billing requires connectivity.

**Validation & Limits:** Valid payment details required to upgrade; downgrade takes effect at the end of the current billing period, never mid-period without explanation.

**Access:** Only Maya (Organiser) has access to billing per the Access Matrix; Sam and Jordan have no billing access. An unauthorized visitor cannot view or change billing.

**Communications:** Sends upgrade/downgrade confirmations and a payment-failure grace-period notice to the organiser.

**Data Notes:** Captured: chosen tier and billing details. Displayed: current tier, what it includes, and billing history to the organiser. Derived: none. Source: organiser input at upgrade/downgrade time.

**Interactions:** Gates AI Weekly Dinner Plan Generation (FEAT-03), Pantry-Aware Suggestions (FEAT-05), and Meal Rating & Preference Learning (FEAT-12), all of which check Subscription tier before running their paid behavior.

**Signals:** subscription_upgraded, subscription_downgraded, subscription_payment_failed, subscription_cancelled.

### Member Onboarding

**ID:** FEAT-15

**Description:** An adult who accepts a household invitation is guided from acceptance to seeing the current plan and grocery list for the first time.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** The brief's growth model depends on invited members having a smooth first experience — "growth is expected to come mostly from households inviting other households" (BRIEF.md, Business Context) is a household-to-household version of the same principle, and a confusing first join would undermine it. Included at MVP alongside Household Invitations (FEAT-09), since an invitation without a guided first-use is only half the feature.

**Connected Entities:** Member Profile (read — the newly created one), Invitation (read — the accepted one)

**Key Capabilities:**
- Land in context — Newly joined member is taken directly to the current plan and grocery list, not a generic empty home screen
- Understand their role — Newly joined member sees a brief explanation of what they can do (view the plan, swap, shop, rate) versus what the organiser manages

**Primary Flows & Alternates:**
- Happy path: invited adult accepts -> lands directly on the current week's plan with a short explanation of their role -> can immediately see and use the grocery list
- No active plan yet: if the household has no plan yet (e.g., organiser has not finished setup), the new member sees a short explanation of that state rather than a broken or empty view
- Re-join: a previously removed member who is re-invited goes through the same onboarding again rather than being silently restored to old data

**States:** Empty: covered above — a household with no plan yet shows an explained empty state, not a blank screen. Loading: N/A — landing in context is immediate on acceptance. Error: N/A — this is a one-time guided landing, not an operation that can fail independently of invitation acceptance itself. Offline-degraded: the landing view degrades to the standard offline plan/list view if there is no connectivity at the moment of acceptance.

**Validation & Limits:** Onboarding is shown exactly once per newly accepted invitation.

**Access:** Applies to any newly accepted Other Adult Member per the Access Matrix; the organiser is never shown this flow since they created the household themselves.

**Communications:** N/A — this is an in-app first-use experience, not a separate notification (the invitation itself, sent by FEAT-09, is the communication).

**Data Notes:** Displayed: the current Weekly Plan and Grocery List, plus a short role explanation. Derived: none. Source: existing household data.

**Interactions:** Depends on Household Invitations & Membership (FEAT-09) for the acceptance event it follows.

**Signals:** member_onboarding_started, member_onboarding_completed, member_onboarding_shown_empty_household.

### Units, Currency & Locale Configuration

**ID:** FEAT-16

**Description:** The household can set the measurement units, currency, and supermarket aisle names that fit where they live, so the plan and grocery list read naturally for US and UK households alike.

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief states directly that "units (cups vs grams), currency and supermarket aisle names must be configurable, not hard-coded" for the US and UK launch markets (BRIEF.md, Scale & Non-Functional Expectations). Included at MVP because both markets are targeted from launch, not added later.

**Connected Entities:** Household (update — locale settings)

**Key Capabilities:**
- Choose measurement units — Household sets cups/oz or grams/ml as its default
- Choose currency — Household sets its currency for budget and cost display
- Customize aisle names — Household adjusts the aisle groupings on its grocery list to match how its local store is laid out

**Primary Flows & Alternates:**
- Happy path: organiser sets units, currency, and reviews default aisle names during setup -> the plan and grocery list consistently reflect those choices thereafter
- Later change: organiser changes units or currency after setup; existing budget figures and recipe quantities are converted for display rather than left inconsistent
- Custom aisle renaming: household renames or reorders an aisle grouping to match their specific local store; the change applies to future grocery lists

**States:** Empty: N/A — every household has a default locale configuration from creation (based on setup-time input). Loading: N/A — changes save instantly. Error: a failed save keeps the prior setting active rather than leaving an inconsistent state. Offline-degraded: locale settings are viewable offline; changes require connectivity to save.

**Validation & Limits:** Units limited to a supported set (e.g., cups/oz, grams/ml); currency limited to a supported set covering at least USD and GBP at launch; aisle names are free text with a reasonable length limit.

**Access:** Only Maya (Organiser) can change these settings per the Household Setup row of the Access Matrix; Sam and Jordan see the results but cannot change them. An unauthorized visitor cannot see or change locale settings.

**Communications:** N/A — configuration is a self-initiated, in-app action with no notifications.

**Data Notes:** Captured: unit system, currency, and aisle name customizations. Displayed: consistently across the plan, recipes, and grocery list. Derived: converted quantity/cost displays when units or currency change. Source: organiser input.

**Interactions:** Read by AI Weekly Dinner Plan Generation (FEAT-03) for cost display, Recipe Library (FEAT-08) for quantity display, and Shared Grocery List (FEAT-06) for aisle grouping.

**Signals:** locale_units_set, locale_currency_set, aisle_name_customized.

### Older-Kid Dinner Voting

**ID:** FEAT-17

**Description:** Older kids can vote on which dinner they'd prefer among the week's options, giving them a voice in the plan without needing a full account.

**Priority:** Important

**Phase:** Later

**Type:** User-Facing

**Rationale:** The brief names this directly — "older kids want to vote on dinners" (BRIEF.md, Target Users & Roles) — but also leaves kids' representation as an explicit open question, with the founder leaning toward "possibly a limited login for older kids later" (BRIEF.md, Open Questions). Phased to Later because it depends on resolving that open question about how older kids are represented at all.

**Connected Entities:** Member Profile (read — the older-kid profile), Weekly Plan (read)

**Key Capabilities:**
- Vote on an option — Older kid picks a preferred dinner among a small set of safe alternatives for a given night
- See the outcome — Older kid sees which option was chosen once the organiser finalizes or the vote naturally resolves

**Primary Flows & Alternates:**
- Happy path: older kid is shown a small set of already-safety-checked options for a night -> votes for a preference -> the outcome is reflected in the plan
- No vote cast: a night with no vote cast simply proceeds with the AI's original suggestion — voting is an enhancement, never a blocker to having a plan
- Tie or no consensus: when multiple kids vote for different options, the organiser sees the split and makes the final call rather than the app arbitrarily deciding

**States:** Empty: a night with no voting round open shows nothing extra — this is normal. Loading: N/A — voting is a simple, instant tap. Error: a failed vote submission retries automatically. Offline-degraded: a vote cast offline is held locally and synced once connectivity returns.

**Validation & Limits:** Voting options are limited to a small set (2-3) that have already passed the allergy safety check; one vote per older-kid profile per voting round.

**Access:** Only older-kid profiles with the (Later-phase) limited login can vote, per the brief's open question; Jordan as a young kid profile with no login has no access to this feature at all, consistent with the Access Matrix's None entry for kid profiles on features requiring their own action. Maya retains final say when votes conflict.

**Communications:** N/A — voting is a self-initiated, in-app action; no separate notification is sent for this Later-phase feature.

**Data Notes:** Captured: each older kid's vote. Displayed: the vote outcome to the household. Derived: none beyond simple tallying. Source: older-kid profile input.

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) for the options it offers and Dietary Rules & Allergy Safety Engine (FEAT-02) to ensure every option is safe.

**Signals:** dinner_vote_opened, dinner_vote_cast, dinner_vote_resolved.

### Account & Data Management

**ID:** FEAT-18

**Description:** The organiser can review, export, or permanently delete the household's data, and manage the account itself — including the minimal data held for kid profiles.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** The brief's privacy stance is explicit and repeated: "children's data is minimal, parent-controlled and never used for anything but the family's own plan" and "no ads, ever, and no selling of family data" (BRIEF.md, Constraints: Privacy). A stated privacy promise is not credible without a concrete way to exercise it, so this is included at MVP rather than deferred.

**Connected Entities:** Household (delete), Member Profile (delete), Dietary Rule (delete)

**Key Capabilities:**
- Export household data — Organiser downloads a copy of the household's plans, ratings, and settings
- Delete a member profile — Organiser removes a member (including a kid profile) and their associated data
- Delete the household — Organiser permanently deletes the entire household and all its data

**Primary Flows & Alternates:**
- Happy path: organiser requests an export -> receives a complete, readable copy of the household's data within a reasonable time
- Member removal: organiser removes a kid or adult profile; that profile's dietary rules and ratings are deleted, and future plans no longer account for them
- Full deletion: organiser requests household deletion -> is shown clearly what will be lost and asked to confirm -> all household data, including every member profile, is permanently removed

**States:** Empty: N/A — this feature always has content to act on once a household exists. Loading: an export or deletion request shows clear progress rather than an indefinite wait. Error: a failed export or deletion is retried automatically or clearly reported, never left in an ambiguous half-deleted state. Offline-degraded: requesting export or deletion requires connectivity; the request queues if made offline and completes once reconnected.

**Validation & Limits:** Household deletion requires explicit confirmation of an irreversible action; export requests are rate-limited to a reasonable frequency to prevent abuse.

**Access:** Only Maya (Organiser) has access to export or delete household data per the Access Matrix; Sam and Jordan cannot request either. An unauthorized visitor has no access. Riley (Operator, support) cannot see or trigger deletion or export.

**Communications:** Sends a confirmation once an export is ready or a deletion completes.

**Data Notes:** Captured: none new. Displayed: an export file and deletion confirmations. Derived: the export file itself, compiled from all household records. Source: all existing household data.

**Interactions:** Reads and can remove data from Household Setup & Member Profiles (FEAT-01), and cascades to remove related Weekly Plan, Grocery List, and Rating data on full deletion.

**Signals:** data_export_requested, data_export_completed, member_deleted, household_deleted.

## Nice-to-Have Features

### Weekly Plan History

**ID:** FEAT-19

**Description:** Household can look back at previous weeks' plans and grocery lists.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** User-Facing

**Rationale:** Not named directly in the brief, but a natural extension once several weeks of plans exist — useful for households wanting to repeat a past week or remember what they ate. Nice-to-Have because the core weekly loop (plan, swap, shop) functions completely without it. Phased to v1, once households have accumulated enough history for it to be useful.

**Connected Entities:** Weekly Plan (read), Grocery List (read)

**Key Capabilities:**
- Browse past weeks — Household navigates back through previously completed weekly plans
- Re-use a past plan — Household can copy a liked past week's plan into a future week as a starting point

**Primary Flows & Alternates:**
- Happy path: household opens history -> browses back through completed weeks -> optionally copies one into a future week
- No history yet: a household in its first week sees a plain "no past weeks yet" message rather than an empty grid
- Re-use with changed household: copying a past plan into a new week still re-runs the current allergy safety check, in case dietary rules changed since then

**States:** Empty: covered above — a clear explanatory message rather than a bare screen. Loading: past weeks load within a couple of seconds. Error: a failed history load offers a retry, never a silent blank page. Offline-degraded: previously viewed history remains available offline; browsing further back may require connectivity.

**Validation & Limits:** History is retained for the life of the household account; no explicit cap on how far back a household can browse.

**Access:** Maya and Sam have Full and View access respectively per the Access Matrix's Weekly Plan row; Jordan has View access. An unauthorized visitor cannot see history.

**Communications:** N/A — browsing history is self-initiated with no notifications.

**Data Notes:** Displayed: past Weekly Plans and Grocery Lists. Derived: none beyond retrieval. Source: archived Weekly Plan and Grocery List records.

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) and Shared Grocery List (FEAT-06) for the archived data it displays; re-use depends on Dietary Rules & Allergy Safety Engine (FEAT-02) re-checking copied plans.

**Signals:** plan_history_opened, past_plan_reused.

### Online Grocery Ordering Handoff

**ID:** FEAT-20

**Description:** The household's grocery list can be handed off to an online grocery ordering capability for delivery, instead of shopping in person.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** Platform

**Rationale:** The brief names this directly as "desirable later, not v1" (BRIEF.md, Ecosystem & Integrations: "Online grocery ordering (e.g. Instacart, Tesco)"). Phased to Later per the brief's own explicit timing; it is documented here rather than only as a deferral note because it is a genuine feature the product will eventually need, not merely an idea.

**Connected Entities:** Grocery List (read), Grocery List Item (read)

**Key Capabilities:**
- Hand off the list — Household sends its current grocery list to an online-ordering capability for fulfillment
- See handoff status — Household sees whether the handoff succeeded and can return to in-person shopping if it did not

**Primary Flows & Alternates:**
- Happy path: household chooses to hand off the list -> the ordering capability receives it -> household sees confirmation
- Handoff failure: if the ordering capability cannot accept the list, the household is told clearly and can fall back to normal in-app shopping without losing the list
- Partial availability: some items on the list may not be available through online ordering; those are called out so the household knows what still needs an in-person trip

**States:** Empty: N/A — handoff only applies once a grocery list exists. Loading: handoff shows a brief, explained progress indicator. Error: a failed handoff falls back cleanly to the standard in-app list, per above. Offline-degraded: handoff requires connectivity; the standard grocery list remains fully usable offline regardless.

**Validation & Limits:** Handoff is available only where an online-ordering capability is available for the household's region.

**Access:** Maya and Sam have Full access to initiate a handoff per the Access Matrix's Grocery List row; Jordan has no access. An unauthorized visitor cannot initiate a handoff.

**Communications:** Sends a handoff confirmation or failure notice to whoever initiated it.

**Data Notes:** Displayed: handoff status and any items unavailable through online ordering. Derived: none beyond the handoff attempt itself. Source: existing Grocery List data plus the ordering capability's response.

**Interactions:** Depends on Shared Grocery List (FEAT-06) for the data it hands off.

**Signals:** grocery_handoff_initiated, grocery_handoff_succeeded, grocery_handoff_failed.

### Family Calendar Sync

**ID:** FEAT-21

**Description:** The week's dinners can appear on the household's existing family calendar, so meal plans show up alongside everything else the family has scheduled.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** Platform

**Rationale:** The brief names this directly as "a nice-to-have for showing dinner on the family calendar, not v1" (BRIEF.md, Ecosystem & Integrations). Phased to Later exactly per the brief's own stated timing.

**Connected Entities:** Weekly Plan (read)

**Key Capabilities:**
- Sync the week's dinners — Household connects a calendar capability so each night's dinner appears as an entry
- Keep it current — A swapped meal updates the corresponding calendar entry automatically

**Primary Flows & Alternates:**
- Happy path: household connects a calendar capability -> each night's planned dinner appears as a calendar entry, kept current through the week
- Swap after sync: a meal swap updates the existing calendar entry rather than creating a duplicate
- Disconnected calendar: if the calendar connection is lost, the in-app plan is entirely unaffected — this integration is additive only

**States:** Empty: N/A — sync only applies once a household has connected a calendar. Loading: N/A — sync happens automatically in the background once connected. Error: a failed sync attempt retries automatically and does not affect the in-app plan. Offline-degraded: sync requires connectivity; the in-app plan remains available regardless.

**Validation & Limits:** One calendar connection per household at a time.

**Access:** Only Maya (Organiser) can connect or disconnect the calendar capability per the Access Matrix's Household Setup row; Sam and Jordan see the resulting calendar entries only outside the product, on the calendar itself. An unauthorized visitor has no access.

**Communications:** N/A — calendar entries are the integration's own output; this feature sends no separate notifications.

**Data Notes:** Displayed: N/A within the product beyond a connection status. Derived: calendar entry content, computed from Weekly Plan data. Source: AI Weekly Dinner Plan Generation (FEAT-03) and One-Tap Meal Swap (FEAT-04) output.

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) and One-Tap Meal Swap (FEAT-04) for the content it syncs.

**Signals:** calendar_connected, calendar_sync_completed, calendar_sync_failed, calendar_disconnected.

### Operator Read-Only Support Access

**ID:** FEAT-22

**Description:** The founder, acting as operator, can view a household's setup and plan in read-only form to help diagnose a reported problem — and nothing more.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** Platform

**Rationale:** The brief names this directly: "the founder, as operator, needs only read-only support access to help a household. Nothing more." (BRIEF.md, Target Users & Roles). Nice-to-Have and phased to v1 because the product can launch and be supported manually at very small scale; a dedicated read-only view becomes worth building once household volume makes ad hoc support impractical.

**Connected Entities:** Household (read), Member Profile (read — excluding kid profile dietary detail beyond what a specific report requires), Weekly Plan (read)

**Key Capabilities:**
- View household setup — Operator sees a household's setup and plan to diagnose a specific reported issue
- Nothing more — Operator cannot edit any household data, view billing detail beyond plan tier, or access kid profile data beyond what a specific safety report requires

**Primary Flows & Alternates:**
- Happy path: a household reports a problem -> operator opens read-only access to that specific household -> diagnoses the issue -> access is not used to make any change
- Scope creep prevention: the view itself has no edit controls at all, so there is no path by which read-only access could accidentally become a change
- Kid data restriction: a support case that does not involve a child's allergy safety never surfaces kid profile dietary detail, consistent with the brief's children's-privacy constraint

**States:** Empty: N/A — this feature has no content of its own beyond an existing household's data. Loading: N/A — a lightweight read-only view with no heavy computation. Error: N/A — a read failure simply shows nothing rather than partial or stale data. Offline-degraded: N/A — this is an operator-side tool requiring connectivity by nature.

**Validation & Limits:** Access is scoped to one household at a time, opened only in response to a specific support case; no bulk or cross-household browsing.

**Access:** Riley (Operator, support) has View-only access to Household Setup, Weekly Plan, Pantry, Grocery List, Recipe Library, and Ratings per the Access Matrix, and no access at all to Kid Profile Data, Meal Swap, Notification Prefs, or edit actions of any kind. No household member has access to this feature themselves — it exists only for the operator.

**Communications:** N/A — this is an internal support tool with no household-facing notifications.

**Data Notes:** Displayed: existing household data in read-only form. Derived: none. Source: existing household records.

**Interactions:** Reads from Household Setup & Member Profiles (FEAT-01), AI Weekly Dinner Plan Generation (FEAT-03), and Shared Grocery List (FEAT-06) without modifying any of them.

**Signals:** operator_support_view_opened, operator_support_view_closed.

## Feature Interaction Summary

| Feature | Depends On |
|---------|------------|
| FEAT-01 Household Setup & Member Profiles | None |
| FEAT-02 Dietary Rules & Allergy Safety Engine | FEAT-01 (dietary rule data) |
| FEAT-03 AI Weekly Dinner Plan Generation | FEAT-01, FEAT-02, FEAT-05, FEAT-08, FEAT-10, FEAT-12, FEAT-14 |
| FEAT-04 One-Tap Meal Swap | FEAT-03, FEAT-02 |
| FEAT-05 Pantry-Aware Suggestions | None |
| FEAT-06 Shared Grocery List | FEAT-03, FEAT-04, FEAT-05 |
| FEAT-07 Weekly Plan Ready Notification | FEAT-03 |
| FEAT-08 Recipe Library (Starter Recipes) | FEAT-02 (badges) |
| FEAT-09 Household Invitations & Membership | FEAT-01 |
| FEAT-10 Recipe Import from Web Link | FEAT-08, FEAT-02 (badges) |
| FEAT-11 Leftover Rollover to Lunches | FEAT-03, FEAT-04 |
| FEAT-12 Meal Rating & Preference Learning | FEAT-03, FEAT-01 (dietary rule updates) |
| FEAT-13 Tonight's Dinner Reminder | FEAT-03, FEAT-04 |
| FEAT-14 Subscription & Billing Management | None |
| FEAT-15 Member Onboarding | FEAT-09 |
| FEAT-16 Units, Currency & Locale Configuration | None |
| FEAT-17 Older-Kid Dinner Voting | FEAT-03, FEAT-02 |
| FEAT-18 Account & Data Management | FEAT-01 |
| FEAT-19 Weekly Plan History | FEAT-03, FEAT-06, FEAT-02 (re-check on reuse) |
| FEAT-20 Online Grocery Ordering Handoff | FEAT-06 |
| FEAT-21 Family Calendar Sync | FEAT-03, FEAT-04 |
| FEAT-22 Operator Read-Only Support Access | FEAT-01, FEAT-03, FEAT-06 |
