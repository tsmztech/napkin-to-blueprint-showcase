---
document_type: feature-dependency-map
produced_by: requirements-architect
status: final
created: 2026-09-26
feature_count: 25
shared_entity_count: 16
cross_feature_rule_count: 20
external_touchpoint_count: 9
---

# Feature Dependency Map

This map is derived from the seven final Stage 2 documents in `.n2b/features/` and covers all 25 features of Plateful, a shared family meal planner for one household per account (BRIEF.md, Vision; Target Users & Roles). Roles named below trace to the Access Matrix in `user-persona.md`: Maya (Organiser), Sam (Other Adult Member), Jordan as a young kid profile with no login (MVP), Jordan as an older kid with a limited login (Later), and Riley (Operator, support — from v1). Feature numbers and Phase values are carried unchanged from `product-features.md`.

## Features

| Number | Slug | Name | Priority | Phase | Type | Depends On | Depended On By |
|--------|------|------|----------|-------|------|------------|----------------|
| FEAT-01 | household-setup-member-profiles | Household Setup & Member Profiles | Core | MVP | User-Facing | -- | FEAT-02, FEAT-03, FEAT-09, FEAT-12, FEAT-18, FEAT-22, FEAT-23, FEAT-24, FEAT-25 |
| FEAT-02 | dietary-rules-allergy-safety-engine | Dietary Rules & Allergy Safety Engine | Core | MVP | Platform | FEAT-01 | FEAT-03, FEAT-04, FEAT-08, FEAT-10, FEAT-17, FEAT-19, FEAT-22, FEAT-23 |
| FEAT-03 | ai-weekly-dinner-plan-generation | AI Weekly Dinner Plan Generation | Core | MVP | User-Facing | FEAT-01, FEAT-02, FEAT-05, FEAT-08, FEAT-10, FEAT-12, FEAT-14, FEAT-16 | FEAT-04, FEAT-06, FEAT-07, FEAT-11, FEAT-12, FEAT-13, FEAT-17, FEAT-19, FEAT-21, FEAT-22 |
| FEAT-04 | one-tap-meal-swap | One-Tap Meal Swap | Core | MVP | User-Facing | FEAT-02, FEAT-03, FEAT-23 | FEAT-06, FEAT-11, FEAT-13, FEAT-21 |
| FEAT-05 | pantry-aware-suggestions | Pantry-Aware Suggestions | Core | MVP | User-Facing | FEAT-14 | FEAT-03, FEAT-06 |
| FEAT-06 | shared-grocery-list | Shared Grocery List | Core | MVP | User-Facing | FEAT-03, FEAT-04, FEAT-05, FEAT-16, FEAT-23 | FEAT-19, FEAT-20, FEAT-22 |
| FEAT-07 | weekly-plan-ready-notification | Weekly Plan Ready Notification | Core | MVP | Platform | FEAT-03 | -- |
| FEAT-08 | recipe-library-starter-recipes | Recipe Library (Starter Recipes) | Core | MVP | User-Facing | FEAT-02 | FEAT-03, FEAT-10, FEAT-23 |
| FEAT-09 | household-invitations-membership | Household Invitations & Membership | Core | MVP | User-Facing | FEAT-01 | FEAT-15, FEAT-18 |
| FEAT-10 | recipe-import-from-web-link | Recipe Import from Web Link | Important | v1 | User-Facing | FEAT-02, FEAT-08 | FEAT-03, FEAT-23 |
| FEAT-11 | leftover-rollover-to-lunches | Leftover Rollover to Lunches | Important | MVP | User-Facing | FEAT-03, FEAT-04 | -- |
| FEAT-12 | meal-rating-preference-learning | Meal Rating & Preference Learning | Important | MVP | User-Facing | FEAT-01, FEAT-03, FEAT-14, FEAT-23 | FEAT-03 |
| FEAT-13 | tonights-dinner-reminder | Tonight's Dinner Reminder | Important | MVP | Platform | FEAT-03, FEAT-04, FEAT-23 | -- |
| FEAT-14 | subscription-billing-management | Subscription & Billing Management | Important | MVP | Lifecycle | -- | FEAT-03, FEAT-05, FEAT-12, FEAT-24 |
| FEAT-15 | member-onboarding | Member Onboarding | Important | MVP | Lifecycle | FEAT-09 | -- |
| FEAT-16 | units-currency-locale-configuration | Units, Currency & Locale Configuration | Important | MVP | Platform | -- | FEAT-03, FEAT-06, FEAT-25 |
| FEAT-17 | older-kid-dinner-voting | Older-Kid Dinner Voting | Important | Later | User-Facing | FEAT-02, FEAT-03 | -- |
| FEAT-18 | account-data-management | Account & Data Management | Important | MVP | Lifecycle | FEAT-01, FEAT-09 | FEAT-22 |
| FEAT-19 | weekly-plan-history | Weekly Plan History | Nice-to-Have | v1 | User-Facing | FEAT-02, FEAT-03, FEAT-06 | -- |
| FEAT-20 | online-grocery-ordering-handoff | Online Grocery Ordering Handoff | Nice-to-Have | Later | Platform | FEAT-06 | -- |
| FEAT-21 | family-calendar-sync | Family Calendar Sync | Nice-to-Have | Later | Platform | FEAT-03, FEAT-04 | -- |
| FEAT-22 | operator-read-only-support-access | Operator Read-Only Support Access | Nice-to-Have | v1 | Platform | FEAT-01, FEAT-02, FEAT-03, FEAT-06, FEAT-18 | -- |
| FEAT-23 | manual-weekly-planning | Manual Weekly Planning | Core | MVP | User-Facing | FEAT-01, FEAT-02, FEAT-08, FEAT-10 | FEAT-04, FEAT-06, FEAT-12, FEAT-13 |
| FEAT-24 | invite-another-household | Invite Another Household | Important | MVP | User-Facing | FEAT-01, FEAT-14 | -- |
| FEAT-25 | weekly-waste-spend-check-in | Weekly Waste & Spend Check-In | Important | MVP | User-Facing | FEAT-01, FEAT-16 | -- |

Depends On is carried from the Feature Interaction Summary in `product-features.md`; FEAT-04's "FEAT-03 or FEAT-23" and FEAT-12's "FEAT-03 or FEAT-23" are recorded as dependencies on both, since either feature can supply the plan. Depended On By is the exact inverse.

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

## Navigation Connections

| From Feature | From Context | To Feature | To Context | Trigger |
|-------------|-------------|------------|-----------|---------|
| FEAT-01 | guided setup | FEAT-16 | units, currency and aisle settings | Setup step "Set units, currency and aisle layout" (First Household Setup, step 4) |
| FEAT-01 | guided setup | FEAT-09 | send invitation | Tap "invite" for a partner during setup (First Household Setup, step 5) |
| FEAT-01 | setup complete confirmation | FEAT-23 | next week's empty nights | Choose "pick this week's dinners" on the free tier (First Household Setup, step 6) |
| FEAT-01 | setup complete confirmation | FEAT-14 | paid-tier overview | Choose "upgrade" (First Household Setup, step 6) |
| FEAT-01 | household settings | FEAT-07 | plan-ready preference and plan-arrival day | Open notification settings (preference toggle lives in setup) |
| FEAT-01 | household settings | FEAT-13 | nightly nudge preference | Open notification settings (preference toggle lives in setup) |
| FEAT-01 | household settings | FEAT-22 | support visit record | Open "support access" record (organiser sees when and why support viewed) |
| FEAT-01 | household settings | FEAT-21 | calendar connection | Tap "connect calendar" (Later) |
| FEAT-07 | "next week's plan is ready" notification | FEAT-03 | new week's plan | Tap the notification (Sunday Plan Review, step 1) |
| FEAT-03 | week plan | FEAT-04 | pending swap suggestions | Review Sam's suggestion and accept (Sunday Plan Review, step 3) |
| FEAT-03 | planned meal | FEAT-04 | safe alternatives list | Tap swap on a meal (Sunday Plan Review, step 4) |
| FEAT-03 | week plan | FEAT-17 | vote split for a night | Review an older kid's vote (Later; Sunday Plan Review, step 3) |
| FEAT-03 | planned meal | FEAT-08 | recipe detail | Tap a meal to open its recipe (Recipe Library cross-reference from plan) |
| FEAT-03 | planned meal | FEAT-02 | report a safety concern | Tap "report a safety concern" (Allergy-Safe Swap Recovery, step 5) |
| FEAT-02 | safety concern acknowledgement | FEAT-04 | safe replacement list | Pick a replacement for the removed meal (Allergy-Safe Swap Recovery, step 5) |
| FEAT-03 | week plan | FEAT-06 | shared grocery list | Open the list from the approved week (Sunday Plan Review, step 5) |
| FEAT-03 | week plan | FEAT-11 | leftover lunch on next day | Confirm or skip the leftover lunch (Weeknight Dinner, step 4) |
| FEAT-03 | week plan | FEAT-12 | rating prompt on a cooked meal | Rate after dinner (Weeknight Dinner, step 3) |
| FEAT-03 | week plan (top card) | FEAT-25 | weekly check-in card | Answer the check-in shown at the top of the plan (End-of-Week Check-In, entry) |
| FEAT-04 | safe alternatives list | FEAT-08 | ineligible-recipe explanation | Search the library directly for a recipe not offered (Allergy-Safe Swap Recovery, failure variant) |
| FEAT-13 | "Tonight: …" nudge | FEAT-03 | tonight's dinner in the plan | Tap the nudge (Weeknight Dinner, step 1) |
| FEAT-23 | a night in next week | FEAT-08 | recipe search and pick list | Tap a night, then search (Free-Tier Manual Week, step 2) |
| FEAT-23 | week being built | FEAT-04 | pending pick suggestions | Accept Sam's suggested pick (Free-Tier Manual Week, step 3) |
| FEAT-23 | week being built | FEAT-06 | shared grocery list | See the list fill from picks (Free-Tier Manual Week, step 5) |
| FEAT-06 | grocery list item | FEAT-05 | pantry list | Tap "already have it" and add to pantry |
| FEAT-06 | grocery list | FEAT-20 | ordering handoff | Tap "hand off list" (Later) |
| FEAT-10 | review extracted recipe | FEAT-08 | household recipe library | Save the imported recipe (Recipe Import & Pantry Update, step 3) |
| FEAT-08 | recipe library | FEAT-10 | paste a link | Tap "import from link" (Recipe Import & Pantry Update, step 1) |
| FEAT-05 | pantry list | FEAT-03 | next week's plan pantry callout | Open the plan that uses logged items (Recipe Import & Pantry Update, step 5) |
| FEAT-09 | invitation link | FEAT-15 | first-use landing | Accept the invitation (Invite & Join Household, step 2–3) |
| FEAT-15 | first-use landing | FEAT-03 | current week's plan | Land in context on the current plan (Invite & Join Household, step 3) |
| FEAT-15 | first-use landing | FEAT-23 | current manually built week | Land in context on a free-tier household's plan |
| FEAT-15 | first-use landing | FEAT-06 | shared grocery list | Open the list from the landing (Invite & Join Household, step 3) |
| FEAT-14 | upgrade confirmation | FEAT-03 | AI plan (first plan on its way) | Features unlock after subscribing (Upgrading & Managing the Account, step 3) |
| FEAT-14 | account settings | FEAT-18 | export household data | Request an export (Upgrading & Managing the Account, step 4) |
| FEAT-18 | account settings | FEAT-22 | support request raised | Contact support (general support contact) |
| FEAT-25 | check-in trend card | FEAT-24 | invite another family | Tap "invite another family" (End-of-Week Check-In, step 3) |
| FEAT-24 | referral welcome page | FEAT-01 | new household setup | Follow the link and start setup (End-of-Week Check-In, step 4) |
| FEAT-19 | past week | FEAT-23 | future week pre-filled | Copy a past week into a future week (v1) |

## Cross-Feature Business Rules

| Rule ID | Description | Affected Features | Authority |
|---------|-------------|-------------------|-----------|
| XBR-01 | Every path onto the plan — AI generation, manual picks, swaps and swap/pick suggestions, voting options, and re-used past weeks — passes the same app-enforced allergy and religious-rule check before anyone sees it; the check fails closed (a recipe with incomplete ingredient data is excluded, never shown unchecked), and every shown meal carries the "checked against allergies" badge and "always check labels" disclaimer | FEAT-02, FEAT-03, FEAT-04, FEAT-08, FEAT-10, FEAT-17, FEAT-19, FEAT-23 | FEAT-02 (owns the safety determination) |
| XBR-02 | A new or tightened hard rule (allergy or religious rule) takes effect on the current week immediately: remaining dinners are re-checked, any that now fail are flagged and removed, safe alternatives are offered through swap, the grocery list updates, and the organiser is told | FEAT-01, FEAT-02, FEAT-04, FEAT-06 | FEAT-02 (runs the re-check; FEAT-01 owns the rule data that triggers it) |
| XBR-03 | The grocery list is always derived from the current plan: every pick, change, swap, accepted suggestion, or safety removal updates the list immediately, ingredients repeated across dinners combine into one line sized for the household, and no member ever sees a week's plan without its matching list | FEAT-03, FEAT-04, FEAT-02, FEAT-06, FEAT-23 | FEAT-06 (owns the Grocery List) |
| XBR-04 | Logged pantry items are left off the week's grocery list on both tiers, and marking a list item "already have it" can add it to the pantry in the same tap; only the paid tier weights plan selection toward pantry items | FEAT-05, FEAT-06, FEAT-03, FEAT-14 | FEAT-05 (owns Pantry Item data) |
| XBR-05 | Tier gating: AI plan generation, pantry-weighted suggestions, and learning from ratings are paid; free and downgraded households plan through Manual Weekly Planning; rating, pantry logging, and the shared list stay free; a downgrade or lapsed payment never removes any past plan, rating, recipe, pantry item, or list | FEAT-14, FEAT-03, FEAT-05, FEAT-12, FEAT-23 | FEAT-14 (owns the Subscription) |
| XBR-06 | Other adult members suggest swaps and picks rather than making them; the organiser accepts or declines each with one tap; at most one open suggestion per member per night; a suggestion not answered before its night lapses and the suggester is told the outcome | FEAT-04, FEAT-23, FEAT-03 | FEAT-04 (manages the Swap Suggestion entity) |
| XBR-07 | Only the organiser approves the week's plan, once per week; later changes happen through swaps; a plan not approved by the start of the week is adopted as proposed so the household is never without a plan | FEAT-03, FEAT-04, FEAT-07 | FEAT-03 (owns the Weekly Plan approval state) |
| XBR-08 | A safety-concern report removes the meal from the household's plan at once, excludes the recipe for that household while the report is open, drops its ingredients from the list, offers safe alternatives, reaches the operator for review, and tells the household the outcome | FEAT-02, FEAT-04, FEAT-06, FEAT-22, FEAT-01 | FEAT-02 (creates safety Support Requests and owns exclusion) |
| XBR-09 | A same-day swap after the nightly nudge was sent triggers at most one follow-up correction naming the new dinner | FEAT-04, FEAT-13 | FEAT-13 (owns the nudge and its correction) |
| XBR-10 | A leftover lunch links to exactly one source dinner and a following day no more than two days later; swapping or removing the source dinner updates or withdraws its leftover suggestion | FEAT-11, FEAT-03, FEAT-04, FEAT-23 | FEAT-11 (creates the leftover-lunch Planned Meal) |
| XBR-11 | The household's units, currency, and aisle names apply consistently everywhere they appear — plan cost and weekly total, recipe quantities, grocery-list aisle grouping and quantities, budget, and check-in spend — and a later change converts existing figures for display rather than leaving them inconsistent | FEAT-16, FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-23, FEAT-25 | FEAT-16 (owns locale settings) |
| XBR-12 | The "plan ready" message is sent at most once per household per week, only on generation completion, at the organiser-chosen arrival day and time; the plan's in-app availability never depends on notification delivery; email is the fallback where device notifications are unavailable | FEAT-03, FEAT-07 | FEAT-07 (owns plan-ready delivery) |
| XBR-13 | Each member controls their own plan-ready and nightly-nudge preferences, held on their Member Profile and set within household settings; neither kid row receives notifications | FEAT-01, FEAT-07, FEAT-13 | FEAT-01 (hosts the Member Profile and preference toggles) |
| XBR-14 | Operator support access opens only for one household with an open Support Request, is strictly read-only, never shows payment details or kid profile data beyond the allergy details in a specific safety report, closes when the request is resolved, and every visit is recorded where the organiser can see it | FEAT-22, FEAT-02, FEAT-18, FEAT-01 | FEAT-22 (owns support access) |
| XBR-15 | The household always has exactly one organiser: an organiser must hand over the role to an active adult (who accepts) or delete the household before leaving or deleting their own account | FEAT-09, FEAT-18, FEAT-01 | FEAT-09 (owns organiser hand-over) |
| XBR-16 | Removing a member deletes their dietary rules and ratings and future plans stop accounting for them; a member who leaves on their own keeps ratings only as anonymous influence; household deletion removes all household data, including every member profile, plan, list, and rating, within 30 days | FEAT-18, FEAT-09, FEAT-01, FEAT-12, FEAT-03, FEAT-06 | FEAT-18 (owns deletion) |
| XBR-17 | A meal rated down repeatedly by the same member becomes a learned soft dislike on that member's dietary rules; soft dislikes influence selection but never block a suggestion and never override an explicit rule | FEAT-12, FEAT-01, FEAT-02, FEAT-03 | FEAT-12 (creates learned dislikes) |
| XBR-18 | An accepted invitation creates a Member Profile with Other Adult Member access and triggers first-use onboarding exactly once per accepted invitation; a re-invited former member onboards again rather than being restored to old data | FEAT-09, FEAT-15 | FEAT-09 (owns Invitation acceptance) |
| XBR-19 | Imported recipes join the same library and candidate pool as starter recipes, a duplicate link surfaces the existing recipe, and an edited imported recipe must pass the safety check again before it can appear in any plan | FEAT-10, FEAT-08, FEAT-02, FEAT-03, FEAT-23 | FEAT-10 (owns imported Recipes) |
| XBR-20 | A new household set up from an invite link within 30 days is attributed to at most one referring household, never itself; whether it went on to pay is read from its subscription; someone who already has a household is told so and nothing is recorded | FEAT-24, FEAT-01, FEAT-14 | FEAT-24 (owns Household Referral) |

## External Touchpoints

Every row traces to the `## Dependencies` section of `assumptions-constraints.md` (ASMP-30 to ASMP-37). ASMP-37 names two separate Later-phase capabilities and is therefore split into two rows. Integration Specs were back-filled per analysis batch after Brief validation. The final analysis batch (FEAT-22) ran the full coverage check across all 25 validated Briefs: every row has at least one covering Integration spec, and all 15 Integration specs inventoried across the Briefs map to a row here (no new capability category was discovered). FEAT-22 inventories no Integration spec and has no External Touchpoints row of its own. Its support-visit note (FEAT-22.SPEC-009) is in-app. The safety-concern emails, to the operator and for the household's resolution notice, are specified in FEAT-02.SPEC-010.

| Capability Category | Features Involved | Integration Specs |
|---------------------|-------------------|-------------------|
| AI text/plan generation (ASMP-30) | FEAT-03, FEAT-04 | FEAT-03.SPEC-010, FEAT-04.SPEC-007 |
| Device-notification delivery (ASMP-31) | FEAT-07, FEAT-13, FEAT-04, FEAT-23 | FEAT-07.SPEC-005 (device-notification delivery boundary, shared by FEAT-04, FEAT-13, FEAT-23 notifications); FEAT-04 Brief inventories no Integration spec for this capability (its FEAT-04.SPEC-006 notifications rely on it via FEAT-07); FEAT-23 Brief inventories no Integration spec for this capability (its pick-suggestion alerts are sent by FEAT-04.SPEC-006 via FEAT-07.SPEC-005); FEAT-13 Brief inventories no Integration spec for this capability (its FEAT-13.SPEC-002 nudge and FEAT-13.SPEC-004 same-day correction are delivered via FEAT-07.SPEC-005) |
| Transactional email (ASMP-32) | FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18 | FEAT-01.SPEC-017 (account & recovery), FEAT-02.SPEC-010 (safety reports), FEAT-07.SPEC-006 (plan-ready email fallback), FEAT-14.SPEC-012 (billing confirmations & grace-period notice), FEAT-18.SPEC-012 (export-ready, deletion-completed & support-acknowledgement email) |
| Payment processing (ASMP-33) | FEAT-14 | FEAT-14.SPEC-009 (payment methods, charges and retries, renewal outcome events) |
| Recipe/food-content data (ASMP-34) | FEAT-08, FEAT-02 | FEAT-08.SPEC-004 (starter recipe content seeding & maintenance); FEAT-02 Brief inventories no Integration spec for this capability (it consumes and validates ingredient completeness; ingestion owned by FEAT-08) |
| Real-time data synchronization (ASMP-35) | FEAT-06, FEAT-03, FEAT-04, FEAT-23 | FEAT-03.SPEC-011 (live plan sync), FEAT-06.SPEC-005 (live grocery list sync and offline reconciliation); FEAT-04 Brief inventories no Integration spec for this capability (relies on it via FEAT-06); FEAT-23 Brief inventories no Integration spec for this capability (live propagation of manual picks relies on FEAT-06.SPEC-005 / FEAT-03.SPEC-011) |
| Web-page recipe extraction (ASMP-36, v1) | FEAT-10 | FEAT-10.SPEC-005 (web page recipe extraction) |
| Online grocery ordering (ASMP-37, Later) | FEAT-20 | FEAT-20.SPEC-002 (online grocery ordering handoff: list payload out, acceptance/rejection and item-availability response in) |
| Family calendar (ASMP-37, Later) | FEAT-21 | FEAT-21.SPEC-002 (family calendar connect/disconnect, per-night entry create/update, inbound sync-outcome and disconnect events) |
