# Part F — Data Model

This part is the product's data vocabulary — the entities behind every feature; the physical schema is Part G3.

## Domain Entity Inventory

### Entity: Household
- **Description:** The shared account for one family — its members, dietary rules, budget, schedule, locale settings (units, currency, aisle names), plan-arrival day, and subscription tier all hang off it. There is one household per account in v1. [MODIFIED: locale settings and plan-arrival day named explicitly because FEAT-16 and FEAT-07 capture them and the entity-coverage audit's inverse check requires every captured setting to have a home]
- **Lifecycle:** Created -> Active -> (optionally) Closed/Deleted
- **Created by:** Household Setup & Member Profiles (FEAT-01)
- **Managed by:** Household Setup & Member Profiles (FEAT-01), Units, Currency & Locale Configuration (FEAT-16), Subscription & Billing Management (FEAT-14), Account & Data Management (FEAT-18)
- **Referenced by:** Nearly every feature in the product, including Invite Another Household (FEAT-24), which records which household referred a new one

### Entity: Member Profile
- **Description:** One person in the household — organiser, other adult member, or kid profile — with their own dietary rules, dislikes, and (for adults, and for older kids once the Later-phase limited login exists) their own login and personal notification preferences. A kid profile holds only a first name or nickname, an age band, its dietary rules, and the organiser's parental-consent confirmation. [AUDIT-ADDED: 3 -- inverse entity check: notification preferences (FEAT-07, FEAT-13) and the parental-consent confirmation (FEAT-01) are captured data that needed an owning entity]
- **Lifecycle:** Invited -> Active -> (optionally) Left or Removed
- **Created by:** Household Setup & Member Profiles (FEAT-01), Household Invitations & Membership (FEAT-09)
- **Managed by:** Household Setup & Member Profiles (FEAT-01), Household Invitations & Membership (FEAT-09) (leaving, organiser hand-over), Account & Data Management (FEAT-18) (removal, own-account management)
- **Referenced by:** Dietary Rules & Allergy Safety Engine (FEAT-02), AI Weekly Dinner Plan Generation (FEAT-03), Meal Rating & Preference Learning (FEAT-12), Older-Kid Dinner Voting (FEAT-17)

### Entity: Dietary Rule
- **Description:** A hard or soft constraint tied to a Member Profile — an allergy, a religious rule (e.g., halal), a per-person vegetarian setting, or a learned dislike.
- **Lifecycle:** Created -> Active -> (optionally) Edited or Removed
- **Created by:** Household Setup & Member Profiles (FEAT-01)
- **Managed by:** Household Setup & Member Profiles (FEAT-01), Meal Rating & Preference Learning (FEAT-12) (for learned dislikes)
- **Referenced by:** Dietary Rules & Allergy Safety Engine (FEAT-02), AI Weekly Dinner Plan Generation (FEAT-03)

### Entity: Weekly Plan
- **Description:** The set of seven dinners (and their leftover-to-lunch links) for one household for one week — proposed by the AI on the paid tier or picked by hand on either tier.
- **Lifecycle:** Generated or Started manually -> Reviewed -> Approved by the organiser (or adopted as proposed when the week begins) -> Active (through the week) -> Archived [AUDIT-ADDED: 1 -- journey walk: BRIEF.md states the organiser "approve[s] the weekly plan", so approval is an explicit lifecycle state]
- **Created by:** AI Weekly Dinner Plan Generation (FEAT-03), Manual Weekly Planning (FEAT-23)
- **Managed by:** AI Weekly Dinner Plan Generation (FEAT-03), Manual Weekly Planning (FEAT-23), One-Tap Meal Swap (FEAT-04)
- **Referenced by:** Weekly Plan Ready Notification (FEAT-07), Shared Grocery List (FEAT-06), Leftover Rollover to Lunches (FEAT-11), Tonight's Dinner Reminder (FEAT-13), Older-Kid Dinner Voting (FEAT-17), Weekly Plan History (FEAT-19)

### Entity: Planned Meal
- **Description:** One dinner slot within a Weekly Plan — a specific night, a specific recipe, its safety badges, and its swap history.
- **Lifecycle:** Proposed or Picked -> Confirmed -> (optionally) Swapped or Removed after a safety concern -> Cooked (the night passes)
- **Created by:** AI Weekly Dinner Plan Generation (FEAT-03), Manual Weekly Planning (FEAT-23), Leftover Rollover to Lunches (FEAT-11) (lunch suggestions)
- **Managed by:** One-Tap Meal Swap (FEAT-04), Manual Weekly Planning (FEAT-23), Dietary Rules & Allergy Safety Engine (FEAT-02) (removal after a safety concern or a mid-week rule change)
- **Referenced by:** Shared Grocery List (FEAT-06), Leftover Rollover to Lunches (FEAT-11), Tonight's Dinner Reminder (FEAT-13), Meal Rating & Preference Learning (FEAT-12)

### Entity: Recipe
- **Description:** A cookable dish — ingredients, steps, cook time, rough cost, and dietary badges — either from the starter library or imported from a web link.
- **Lifecycle:** Added -> Active -> (optionally) Archived
- **Created by:** Recipe Library (Starter Recipes) (FEAT-08) (starter content), Recipe Import from Web Link (FEAT-10) (user-imported)
- **Managed by:** Recipe Library (Starter Recipes) (FEAT-08) (starter recipes are read-only for households), Recipe Import from Web Link (FEAT-10) (households edit or remove their own imported recipes) [AUDIT-ADDED: 3 -- entity coverage: imported recipes had no edit or delete path]
- **Referenced by:** AI Weekly Dinner Plan Generation (FEAT-03), Manual Weekly Planning (FEAT-23), Dietary Rules & Allergy Safety Engine (FEAT-02), Shared Grocery List (FEAT-06)

### Entity: Pantry Item
- **Description:** Something the household says it already has on hand, used to steer plan generation away from buying duplicates.
- **Lifecycle:** Added -> Active -> (optionally) Used/Removed
- **Created by:** Pantry-Aware Suggestions (FEAT-05)
- **Managed by:** Pantry-Aware Suggestions (FEAT-05)
- **Referenced by:** AI Weekly Dinner Plan Generation (FEAT-03), Shared Grocery List (FEAT-06)

### Entity: Grocery List
- **Description:** The one combined, aisle-grouped shopping list generated from the week's plan and shared live by the whole household.
- **Lifecycle:** Generated -> Active (through the week) -> Archived
- **Created by:** Shared Grocery List (FEAT-06) (built from an AI-generated or manually picked week)
- **Managed by:** Shared Grocery List (FEAT-06)
- **Referenced by:** Weekly Plan History (FEAT-19), Online Grocery Ordering Handoff (FEAT-20)

### Entity: Grocery List Item
- **Description:** One line on the Grocery List — an ingredient from the plan or something a household member added directly — with its aisle, quantity, and ticked/unticked state.
- **Lifecycle:** Added -> Unticked -> Ticked -> (list archive) Cleared, with unticked manual items carried into the next week's list [AUDIT-ADDED: 3 -- entity coverage: the draft did not say what happens to unbought manual items when the week's list is archived]
- **Created by:** Shared Grocery List (FEAT-06) (from the plan or manual add)
- **Managed by:** Shared Grocery List (FEAT-06)
- **Referenced by:** Online Grocery Ordering Handoff (FEAT-20)

### Entity: Rating
- **Description:** A household member's thumbs up/down on a cooked meal, used to learn what the family actually likes.
- **Lifecycle:** Created -> Active (feeds future plans)
- **Created by:** Meal Rating & Preference Learning (FEAT-12) (by the member, or by an adult on behalf of a young kid profile)
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
- **Referenced by:** AI Weekly Dinner Plan Generation (FEAT-03), Pantry-Aware Suggestions (FEAT-05), Meal Rating & Preference Learning (FEAT-12), Invite Another Household (FEAT-24) (which referred households go on to pay)

### Entity: Swap Suggestion
- **Description:** A proposed change to one night's dinner raised by an other adult member, waiting for the organiser to accept or decline it. [AUDIT-ADDED: 3 -- inverse entity check: BRIEF.md says other adult members "suggest swaps" while the organiser approves the plan, and those suggestions are captured data that needed an entity]
- **Lifecycle:** Suggested -> Accepted or Declined (or Lapsed once the night has passed)
- **Created by:** One-Tap Meal Swap (FEAT-04), Manual Weekly Planning (FEAT-23)
- **Managed by:** One-Tap Meal Swap (FEAT-04)
- **Referenced by:** AI Weekly Dinner Plan Generation (FEAT-03) (plan review before approval)

### Entity: Dinner Vote
- **Description:** One older kid's vote for a preferred dinner among a small set of safe options for a given night. [AUDIT-ADDED: 3 -- inverse entity check: Older-Kid Dinner Voting (FEAT-17) captures votes that no entity held]
- **Lifecycle:** Voting round opened -> Vote cast -> Round resolved (by tally or by the organiser's final call)
- **Created by:** Older-Kid Dinner Voting (FEAT-17)
- **Managed by:** Older-Kid Dinner Voting (FEAT-17)
- **Referenced by:** AI Weekly Dinner Plan Generation (FEAT-03)

### Entity: Household Referral
- **Description:** The record that a new household was set up through an invite link shared by a member of an existing household. [AUDIT-ADDED: 3 -- inverse entity check: the brief's growth success criterion ("most paying households were invited by another household") needs a referral record no draft entity held]
- **Lifecycle:** Link shared -> Link opened -> New household created (referral recorded) -> (optionally) New household upgrades
- **Created by:** Invite Another Household (FEAT-24)
- **Managed by:** Invite Another Household (FEAT-24)
- **Referenced by:** Subscription & Billing Management (FEAT-14) (whether the referred household became a paying one)

### Entity: Waste & Spend Check-In
- **Description:** A household's optional weekly answer about how much food was thrown away and roughly what was spent on groceries, plus a one-time "typical week before Plateful" baseline. [AUDIT-ADDED: 3 -- inverse entity check: the draft's food-waste success metric depended on self-reported answers that no feature captured]
- **Lifecycle:** Offered -> Answered or Skipped -> (editable until the next week's check-in opens)
- **Created by:** Weekly Waste & Spend Check-In (FEAT-25)
- **Managed by:** Weekly Waste & Spend Check-In (FEAT-25)
- **Referenced by:** N/A — no other feature reads it; it feeds only the household's own trend view and product success measurement

### Entity: Support Request
- **Description:** A household's report to the operator — either a safety concern about a specific meal or a general request for help — together with the record of any time support viewed the household to resolve it. [AUDIT-ADDED: 3 -- inverse entity check: safety-concern reports (FEAT-02) and support contact (FEAT-18) are captured data, and Operator Read-Only Support Access (FEAT-22) needs a household-raised case to act on]
- **Lifecycle:** Raised -> Under review -> Resolved
- **Created by:** Dietary Rules & Allergy Safety Engine (FEAT-02) (safety concerns), Account & Data Management (FEAT-18) (general support contact)
- **Managed by:** Operator Read-Only Support Access (FEAT-22) (status only — it never changes household data)
- **Referenced by:** Household Setup & Member Profiles (FEAT-01) (the organiser sees open requests and support access records)



## Entities Shared Across Features

## Shared Data Entities

Sixteen of the seventeen entities in the Domain Entity Inventory are touched by two or more features. The seventeenth, Waste & Spend Check-In, is created, read, and updated only by Weekly Waste & Spend Check-In (FEAT-25) and is therefore not listed here (its inventory entry states no other feature reads it).

### Household

- **Lifecycle:** Created by FEAT-01. Read by nearly every feature, notably FEAT-03, FEAT-06, FEAT-22, FEAT-24, FEAT-25. Updated by FEAT-01 (name, budget, schedule), FEAT-16 (locale settings), FEAT-07 (plan-arrival day and time), FEAT-14 (subscription tier via the Subscription entity). Deleted by FEAT-18.
- **Fields (functional):**
  - household_name -- required, 1–60 characters
  - organiser -- the one Member Profile holding the organiser role (required; changes hands only through FEAT-09's hand-over)
  - weekly_budget -- rough weekly food budget, a positive amount in the household's currency (required before AI generation; optional during partial setup)
  - weekly_schedule -- which nights are short on time and their time limit (e.g., 30-minute weeknights); absent schedule means no time constraint
  - unit_system -- cups/oz or grams/ml (defaulted at creation, changed through FEAT-16)
  - currency -- from a supported set covering at least USD and GBP at launch
  - aisle_names -- the household's aisle groupings and order, free text with a length limit
  - plan_arrival_day_time -- day of week and a slot from a small set of evening/morning times (Sunday evening by default)
  - status -- Active or Closed/Deleted
- **Relationships:** Has 1–12 Member Profiles (one of them the organiser), one Subscription, one Weekly Plan per week, one Grocery List per week, many Pantry Items, many imported Recipes, and many Support Requests; may be the referring or referred household of a Household Referral. One household per account in v1 (ASMP-16).
- **Contention:** Low — only Maya (Organiser) can modify household settings (Household Setup, Billing, and Account & Data columns: Full) through FEAT-01, FEAT-07, FEAT-16, FEAT-14, and FEAT-18; Sam, both Jordan rows, and Riley have View or None. Concurrent edits can occur only when Maya is signed in on two devices (laptop setup plus phone): resolution is last-write-wins per setting, and a failed save keeps the prior value active rather than leaving a mixed state (FEAT-16 States). Household deletion (FEAT-18) supersedes any in-flight edit.
- **Data Sensitivity:** Personal household data (budget, schedule, locale) — private to household members, never sold or used for advertising (ASMP-14, ASMP-26); exportable and deletable under general personal-data rights, with deleted data removed within 30 days (ASMP-27, FEAT-18). Riley sees it read-only only against an open Support Request (FEAT-22).
- **Source:** Domain Entity Inventory, product-features.md

### Member Profile

- **Lifecycle:** Created by FEAT-01 (organiser, adults, kid profiles) and FEAT-09 (on invitation acceptance). Read by FEAT-02, FEAT-03, FEAT-12, FEAT-15, FEAT-17, FEAT-22. Updated by FEAT-01 (profile details), FEAT-09 (organiser hand-over, leaving), FEAT-18 (own-account management), FEAT-07 and FEAT-13 (each member's own notification preferences). Deleted by FEAT-18 (member removal, own-account deletion, household deletion) and FEAT-09 (member leaves).
- **Fields (functional):**
  - display_name -- first name or nickname (required); adults may also hold an email for sign-in
  - member_type -- Organiser, Other Adult Member, young kid profile (no login), or older kid limited login (Later)
  - sign_in -- email and protected sign-in for adults only; young kid profiles hold none
  - age_band -- kid profiles only (required for kids)
  - parental_consent_confirmation -- kid profiles only; the organiser's confirmation of being parent or guardian (required to create a kid profile)
  - notification_preferences -- per member: plan-ready on/off, nightly nudge on/off (adults only)
  - status -- Invited, Active, Left, or Removed
- **Relationships:** Belongs to one Household; owns zero or more Dietary Rules; authors Ratings, Swap Suggestions, Support Requests, and (older kids, Later) Dinner Votes; an adult member owns one personal household-referral link.
- **Contention:** Maya (Organiser) edits any profile through FEAT-01 and removes members through FEAT-18, while Sam (Other Adult Member) edits his own account (FEAT-18, Own-only), changes his own notification preferences (FEAT-07/FEAT-13), or leaves (FEAT-09). A removal by Maya racing an own-account edit by Sam resolves reject-with-refresh: removal wins and Sam's edit is refused with a clear message. Own-field edits from two devices are last-write-wins. Organiser hand-over requires the recipient's acceptance, so only one organiser exists at any moment.
- **Data Sensitivity:** Kid profiles are children's data under children's-privacy-class protection — only a first name or nickname, an age band, dietary rules, and the consent confirmation; no surname, birth date, photo, or contact detail; parent-controlled and used only for the household's own plan (ASMP-26, ASMP-27, FEAT-01 Validation & Limits). Adult sign-in details and email are personal data requiring account protection and recovery. Riley never sees kid profile data except the allergy details inside a specific safety report (Access Matrix notes).
- **Source:** Domain Entity Inventory, product-features.md

### Dietary Rule

- **Lifecycle:** Created by FEAT-01. Read by FEAT-02, FEAT-03, FEAT-23, FEAT-04 (through FEAT-02). Updated by FEAT-01 (explicit rules) and FEAT-12 (learned soft dislikes). Deleted by FEAT-01 (with explicit confirmation for allergies) and FEAT-18 (cascade on member removal or household deletion).
- **Fields (functional):**
  - member -- the Member Profile the rule belongs to (required)
  - rule_kind -- allergy, religious rule (e.g., halal), per-person vegetarian setting, or dislike
  - strength -- hard (allergy, religious rule) or soft (dislike); vegetarian applies per person with a shared-meal vegetarian option
  - allergen -- from a standard allergen list, optionally a named extra ingredient (required for allergies)
  - origin -- entered by the organiser or learned from ratings
  - change_history -- who changed the rule and when, visible to the organiser
- **Relationships:** Belongs to one Member Profile; read by the safety engine against every ingredient of every candidate Recipe.
- **Contention:** Maya (Organiser) edits rules through FEAT-01 while FEAT-12 may add a learned soft dislike from ratings recorded by Maya, Sam, or (Later) the older-kid login. Resolution is merge: learned dislikes are separate soft entries that never overwrite or weaken an explicit rule, and an explicit organiser edit takes precedence over a learned entry for the same ingredient. An allergy can never be dropped without Maya's explicit confirmation, so no concurrent path can silently remove a hard rule.
- **Data Sensitivity:** Health-adjacent personal data, including children's allergy information — the most sensitive data in the product; children's-privacy-class protection, parent-controlled, used only for the household's own plan, never sold or used for advertising, and no medical or diet advice is derived from it (ASMP-17, ASMP-26, ASMP-27). Change history is kept for trust and any safety investigation (FEAT-01 Data Notes).
- **Source:** Domain Entity Inventory, product-features.md

### Weekly Plan

- **Lifecycle:** Created by FEAT-03 (generated, paid tier) and FEAT-23 (started manually, either tier). Read by FEAT-06, FEAT-07, FEAT-11, FEAT-13, FEAT-15, FEAT-17, FEAT-19, FEAT-21, FEAT-22. Updated by FEAT-03 (approval, auto-adoption), FEAT-23 (picks), FEAT-04 (swaps), FEAT-17 (Later, vote outcome), FEAT-19 (v1, re-use of a past week as a starting point). Deleted by FEAT-18 (household deletion cascade); otherwise Archived when the week ends.
- **Fields (functional):**
  - week -- the calendar week the plan covers (required; planning allowed up to one week ahead)
  - origin -- AI-generated or manually built
  - status -- Generated/Started, Reviewed, Approved, Active, Archived
  - approval -- organiser approval (once per week) or auto-adoption at week start
  - estimated_total -- the week's estimated cost against the household budget, in the household's currency
  - over_budget_note -- shown when no safe week fits the budget
- **Relationships:** Belongs to one Household; contains up to seven dinner Planned Meals plus leftover-lunch Planned Meals; drives one Grocery List; receives Swap Suggestions and (Later) Dinner Votes.
- **Contention:** Maya (Organiser, Weekly Plan Full) changes the plan through FEAT-03 (approve), FEAT-23 (pick, change, clear), and FEAT-04 (swap); Sam (View; Own-only on Meal Swap and Manual Planning) changes it only indirectly when Maya accepts his suggestion; FEAT-03's automatic week-start adoption and FEAT-02's safety removals also write to it. Resolution is reject-with-refresh per night slot (no more than one active swap operation per slot, FEAT-04), and a safety removal always wins over any concurrent change. Approval can be given once per week; adoption at week start applies only if no approval has been recorded.
- **Data Sensitivity:** Household personal data (what the family eats) — private to the household, not sold or used for advertising (ASMP-14, ASMP-26); kept for the life of the account and available after downgrade (SC-18, ASMP-19); exportable and deletable (FEAT-18).
- **Source:** Domain Entity Inventory, product-features.md

### Planned Meal

- **Lifecycle:** Created by FEAT-03 (proposed), FEAT-23 (picked), FEAT-11 (leftover lunch suggestion). Read by FEAT-06, FEAT-12, FEAT-13, FEAT-17, FEAT-21. Updated by FEAT-04 (swap), FEAT-23 (change), FEAT-02 (safety badge; removal after a safety concern or a mid-week rule change), FEAT-11 (leftover lunch confirmed or skipped). Deleted by FEAT-23 (clear a night) and FEAT-02 (removal after a safety concern); cascade on household deletion (FEAT-18).
- **Fields (functional):**
  - night -- the day of the week (required; at most one dinner per night)
  - meal_kind -- dinner, or leftover lunch linked to one source dinner no more than two days earlier
  - recipe -- the chosen Recipe (required)
  - safety_badge -- "checked against allergies" plus the "always check labels" disclaimer
  - vegetarian_option -- whether a shared meal carries a vegetarian variant
  - cook_time and rough_cost -- carried from the recipe, sized for the household
  - pantry_callout -- which logged pantry items this dinner uses
  - status -- Proposed/Picked, Confirmed, Swapped, Removed (safety), Cooked; leftover lunches: Suggested, Eaten, Skipped
  - swap_history -- prior recipes in this slot
- **Relationships:** Belongs to one Weekly Plan; references one Recipe; contributes ingredients to the Grocery List; is the subject of Ratings, Swap Suggestions, a Tonight's Dinner nudge, and (Later) Dinner Votes and calendar entries.
- **Contention:** Maya (Organiser) swaps or changes a slot (FEAT-04, FEAT-23) while Sam (Other Adult Member) or Maya may report a safety concern on it (FEAT-02, Safety Reports: Maya Full, Sam Own-only) and Maya or Sam may mark a leftover lunch eaten or skipped (FEAT-11). Resolution: a safety removal wins over any concurrent change; slot changes are reject-with-refresh with at most one active swap per slot, and a double tap never creates two swaps; leftover status updates are last-write-wins.
- **Data Sensitivity:** Household personal data — private to the household, never sold (ASMP-14, ASMP-26); no children's data held directly.
- **Source:** Domain Entity Inventory, product-features.md

### Recipe

- **Lifecycle:** Created by FEAT-08 (starter content) and FEAT-10 (household imports, v1). Read by FEAT-02, FEAT-03, FEAT-04, FEAT-06, FEAT-23, FEAT-19. Updated by FEAT-10 (households edit their own imported recipes; starter recipes are read-only for households). Deleted (removed from the household's pool) by FEAT-10; otherwise Archived.
- **Fields (functional):**
  - name -- required
  - ingredients -- each with quantity and unit; complete ingredient data is required to pass the safety check (at least one ingredient to save an import)
  - steps -- method text, length-capped
  - cook_time -- required for schedule fit
  - rough_cost -- shown in the household's currency
  - dietary_badges -- computed per household by FEAT-02 when viewed
  - origin -- starter library or imported (with source link)
  - owning_household -- imported recipes only; starter recipes belong to no household
  - prep_requirements -- early-prep needs such as defrosting, used for the nightly nudge
- **Relationships:** Referenced by Planned Meals; imported recipes belong to one Household; starter recipes are shared read-only content for all households.
- **Contention:** Starter recipes have no household-side contention (read-only). Imported recipes can be edited or removed by Maya and Sam (Recipe Library: Full) through FEAT-10 concurrently: resolution is reject-with-refresh on the second save, and every accepted edit is re-checked by FEAT-02 before the recipe can appear in a plan again. Riley has View only.
- **Data Sensitivity:** None — recipe content carries no personal data; an imported recipe's source link and household ownership are household data under the general no-sale posture (ASMP-14).
- **Source:** Domain Entity Inventory, product-features.md

### Pantry Item

- **Lifecycle:** Created by FEAT-05 and by FEAT-06's "already have it" action. Read by FEAT-03 (plan weighting, paid tier) and FEAT-06 (kept off the list, both tiers). Updated by FEAT-05 (marked used). Deleted by FEAT-05 (cleared); cascade on household deletion (FEAT-18).
- **Fields (functional):**
  - item_name -- free text, 1–80 characters (required)
  - added_by -- the member who logged it
  - status -- Active, Used/Removed
  - used_prompt -- after the dinner using it has passed, a one-tap "used it up?" prompt
- **Relationships:** Belongs to one Household; matched against Planned Meal ingredients for pantry callouts and against Grocery List lines to leave them off.
- **Contention:** Maya and Sam (Pantry Input: Full) add and clear items through FEAT-05 and FEAT-06, including offline. Resolution is merge: the same item added twice becomes one entry, and clearing is idempotent. The older-kid login (Later) has Pantry Input None, so its "already have it" on the list does not create a pantry item.
- **Data Sensitivity:** Low — household personal data (what is in the fridge), private to the household and never sold (ASMP-14).
- **Source:** Domain Entity Inventory, product-features.md

### Grocery List

- **Lifecycle:** Created by FEAT-06 (from a generated or manually picked week). Read by FEAT-15, FEAT-19, FEAT-20, FEAT-22. Updated by FEAT-06, including automatic recalculation when FEAT-03, FEAT-23, FEAT-04, FEAT-02, or FEAT-05 change the plan or pantry. Archived by FEAT-06 at week end; deleted by FEAT-18 (household deletion cascade).
- **Fields (functional):**
  - week -- the plan week it serves (required)
  - aisle_grouping -- the household's configured aisle names and order
  - status -- Generated, Active, Archived
- **Relationships:** Belongs to one Household and one Weekly Plan; contains Grocery List Items.
- **Contention:** Plan-derived recalculation (triggered by Maya's picks and swaps, accepted suggestions, safety removals) runs while Maya, Sam, and (Later) the older-kid login edit the list. Resolution is merge: plan-derived lines are recomputed while manual items, ticks, and "already have it" marks are preserved.
- **Data Sensitivity:** Low — household personal data, private to the household and never sold (ASMP-14); kept for the life of the account (SC-18).
- **Source:** Domain Entity Inventory, product-features.md

### Grocery List Item

- **Lifecycle:** Created by FEAT-06 (plan-derived or manual add). Read by FEAT-20 (Later). Updated by FEAT-06 (tick, untick, quantity edit, "already have it"). Deleted by FEAT-06 (removed); cleared on list archive, with unticked manual items carried to next week.
- **Fields (functional):**
  - ingredient_name -- manual items 1–80 characters (required)
  - quantity_and_unit -- combined across dinners, sized for the household, in the household's unit system
  - aisle -- from the household's aisle names
  - origin -- plan-derived or manual (with "added by" member)
  - ticked -- ticked or unticked
- **Relationships:** Belongs to one Grocery List; plan-derived items trace to one or more Planned Meals.
- **Contention:** High — Maya and Sam (Grocery List: Full) and the older-kid login (Later, Full for add and tick) tick, add, edit, and remove items live, including offline in the store. Resolution is merge: ticks are idempotent (a double tick offline counts once), duplicate manual adds for the same ingredient merge into one line, conflicting quantity edits are last-write-wins, and offline changes sync without duplicates or conflicts (ASMP-25).
- **Data Sensitivity:** Low — household personal data, private to the household (ASMP-14); the "added by" label shows a member's name only within the household.
- **Source:** Domain Entity Inventory, product-features.md

### Rating

- **Lifecycle:** Created by FEAT-12 (by the member, or by an adult on behalf of a young kid profile). Read by FEAT-03 (paid-tier learning) and FEAT-22 (diagnosis). Updated by FEAT-12 (changeable until the plan is archived) and FEAT-09 (a leaving member's ratings become anonymous influence). Deleted by FEAT-18 (member removal, household deletion).
- **Fields (functional):**
  - member -- the rating member (required)
  - planned_meal -- the cooked meal (required; one rating per member per meal)
  - value -- thumbs up or thumbs down
  - recorded_by -- the adult who recorded it on a young kid's behalf, where applicable
- **Relationships:** Links one Member Profile to one Planned Meal; repeated down-ratings feed a learned soft dislike in Dietary Rule.
- **Contention:** Maya (Ratings: Full) and Sam (Own-only) each rate for themselves and either can record a young kid's rating; the older-kid login (Later) rates its own. Two adults recording the same young kid's rating for the same meal resolve last-write-wins, since one rating per member per meal is kept and ratings are changeable.
- **Data Sensitivity:** Personal preference data, including children's preferences — individual ratings are never shown broken out to other members (aggregate effect only), used only for the household's own plan (ASMP-26); anonymised when a member leaves, deleted on removal.
- **Source:** Domain Entity Inventory, product-features.md

### Invitation

- **Lifecycle:** Created by FEAT-09. Read by FEAT-15. Updated by FEAT-09 (accepted, revoked, expired, resent).
- **Fields (functional):**
  - contact_detail -- the invitee's contact detail (required)
  - status -- Sent, Accepted, Revoked, Expired (after 14 days)
  - sent_by -- the organiser
- **Relationships:** Belongs to one Household; on acceptance creates one Member Profile with Other Adult Member access; triggers one Member Onboarding.
- **Contention:** Only Maya (Household Invitations: Full) creates, resends, or revokes; the invitee (not yet a member) can only accept via their link. An acceptance racing a revocation or expiry resolves reject-with-refresh: whichever state change lands first wins, and an invitee arriving after revocation sees "this invitation is no longer valid."
- **Data Sensitivity:** Personal data of a non-member (contact detail) — used only to deliver the invitation, never sold or used for marketing (ASMP-14).
- **Source:** Domain Entity Inventory, product-features.md

### Subscription

- **Lifecycle:** Created by FEAT-01 (defaults to free). Read by FEAT-03, FEAT-05, FEAT-12 (tier gating), FEAT-24 (referred household upgrades), FEAT-22 (plan tier only). Updated by FEAT-14 (upgrade, period switch, downgrade, cancel, grace period).
- **Fields (functional):**
  - tier -- free or paid
  - billing_period -- monthly or yearly (switch takes effect at next renewal)
  - billing_state -- Active, Payment failed (7-day grace period), Cancelled (paid until period end), Reverted to free
  - billing_history -- visible to the organiser
- **Relationships:** Belongs to one Household; payment details are held with the payment-processing capability and seen only by the organiser.
- **Contention:** Only Maya (Billing: Full) changes the subscription; renewal outcomes and grace-period expiry arrive from the payment-processing capability at the same time. Resolution is reject-with-refresh: an organiser change submitted against a stale billing state is refused and shown the current state; period-boundary rules (downgrade at period end, grace expiry after 7 days) decide timing.
- **Data Sensitivity:** Payment details are financial personal data — visible only to Maya, never to Sam, kids, or Riley (Riley sees plan tier only); held by the payment-processing capability (ASMP-33, Access Matrix notes).
- **Source:** Domain Entity Inventory, product-features.md

### Swap Suggestion

- **Lifecycle:** Created by FEAT-04 (swap suggestion) and FEAT-23 (pick suggestion). Read by FEAT-03 (plan review before approval). Updated by FEAT-04 (accepted, declined, lapsed).
- **Fields (functional):**
  - suggesting_member -- the other adult member (required)
  - night -- the target slot (required; one open suggestion per member per night)
  - proposed_recipe -- a safety-checked recipe (required)
  - outcome -- Suggested, Accepted, Declined, Lapsed (once the night passes)
- **Relationships:** Targets one Planned Meal slot in one Weekly Plan; raised by one Member Profile.
- **Contention:** Sam (Other Adult Member, Own-only) raises suggestions and Maya (Organiser, Full) accepts or declines them, while the lapse rule closes unanswered ones when the night passes. Resolution is first-decision-wins with reject-with-refresh: a late accept on a lapsed suggestion is refused, and a direct swap by Maya on the same slot supersedes any open suggestion for it.
- **Data Sensitivity:** None — the suggestion holds only a member reference and a recipe choice, private to the household under the general posture (ASMP-14).
- **Source:** Domain Entity Inventory, product-features.md

### Dinner Vote

- **Lifecycle:** Created by FEAT-17 (Later). Read by FEAT-03 (plan review). Updated by FEAT-17 (round resolved by tally or organiser's final call).
- **Fields (functional):**
  - round -- a night and its 2–3 safety-checked options
  - voter -- an older-kid limited login (one vote per round)
  - choice -- the chosen option
  - resolution -- tally result or the organiser's final call
- **Relationships:** Belongs to one Weekly Plan night; cast by an older-kid Member Profile.
- **Contention:** Older-kid logins (Dinner Voting: Own-only) cast votes while Maya (Full) may make the final call. A vote cast after Maya has resolved the round is rejected with refresh, showing the outcome.
- **Data Sensitivity:** Children's data (a child's preference) — minimal, parent-controlled, used only for the household's own plan (ASMP-26, ASMP-27).
- **Source:** Domain Entity Inventory, product-features.md

### Household Referral

- **Lifecycle:** Created by FEAT-24 (recorded when a new household completes setup from a link within 30 days). Read by FEAT-24 (count of families joined) and FEAT-14 (whether the referred household became paying).
- **Fields (functional):**
  - referring_household -- the household whose member shared the link
  - referring_member_link -- the member's one reusable personal link
  - new_household -- the household created from the link
  - created_date -- date setup completed
  - upgraded -- whether the new household went on to pay
- **Relationships:** Links two Households; a new household is attributed to at most one referring household and cannot refer itself.
- **Contention:** None — the record is written once by the system when the new household's setup completes and is never edited by any role; the upgrade flag is derived from the new household's Subscription.
- **Data Sensitivity:** Low — links two households; the unauthorized visitor following a link sees only the inviter's first name, never other household data (FEAT-24 Access); used only for product success measurement, never sold (ASMP-14).
- **Source:** Domain Entity Inventory, product-features.md

### Support Request

- **Lifecycle:** Created by FEAT-02 (safety concern) and FEAT-18 (general support contact). Read by FEAT-01 (organiser sees open requests and support access records) and FEAT-22 (v1). Updated by FEAT-22 (status and access record only).
- **Fields (functional):**
  - kind -- safety concern or general support
  - raised_by -- the reporting adult member
  - planned_meal / recipe -- for safety concerns (required)
  - note -- optional, up to 500 characters for safety concerns; a short description for support contact
  - status -- Raised, Under review, Resolved
  - access_record -- each time support viewed the household, when and why, visible to the organiser
- **Relationships:** Belongs to one Household; a safety concern references one Planned Meal and its Recipe; it is the precondition for any operator support view.
- **Contention:** None — Maya and Sam (Safety Reports: Full / Own-only) raise requests but do not edit them afterwards; only Riley (Operator, Support View: Full) changes status, one household at a time.
- **Data Sensitivity:** May contain children's allergy details (a safety report) — children's-privacy-class protection; Riley may see kid allergy detail only inside a specific safety report; every support view is recorded and visible to the organiser (ASMP-26, ASMP-27, FEAT-22).
- **Source:** Domain Entity Inventory, product-features.md

