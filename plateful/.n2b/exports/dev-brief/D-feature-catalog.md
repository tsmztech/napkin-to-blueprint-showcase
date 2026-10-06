# Part D — Feature Catalog

This part is the full feature catalog by priority tier, followed by how the features depend on each other, including the cross-feature business rules (XBR) that bind them. The Domain Entity Inventory below also anchors Part F, the data-model view.


# Product Features

## Summary

This product includes 25 features: 10 Core, 11 Important, 4 Nice-to-Have. By phase: 19 MVP, 3 v1, 3 Later. By type: 15 User-Facing, 7 Platform, 3 Lifecycle. The product manages 17 domain entities. Core features cover household setup, allergy safety, the AI weekly plan, manual weekly planning for the free tier, swapping, pantry awareness, the shared grocery list, the plan-ready notification, the starter recipe library, and household membership; Important features cover recipe import, leftovers, ratings and learning, the dinner nudge, billing, member onboarding, household-to-household invitations, the weekly waste and spend check-in, localization, older-kid voting, and account/data management; Nice-to-Have features cover plan history, two later-phase integrations, and operator support access. [MODIFIED: counts updated from 22 features / 12 entities after synthesis added 3 audit-evidenced features (FEAT-23, FEAT-24, FEAT-25), moved Meal Rating & Preference Learning (FEAT-12) from v1 to MVP, and added 5 entities surfaced by the entity-coverage audit]

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

## Core Features

### Household Setup & Member Profiles

**ID:** FEAT-01

**Description:** The organiser sets up the household once: who eats with them, each person's allergies and diet, the weekly food budget, and how much time there is on which nights. This is the foundation every other feature reads from.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief states the household "is set up once" before anything else can happen (BRIEF.md, Vision). Without this, there is no household to plan for. MVP: nothing else in the product functions without it. [RESEARCH-INFORMED: added competitor context — independent reviews report that single-profile planners make mixed-diet families "hit walls quickly" and that entering several allergies leaves very few options, so per-person rules in one household are the differentiator (Mealime profile, independent and App Store reviews)]

**Connected Entities:** Household (create, update), Member Profile (create, update), Dietary Rule (create, update)

**Key Capabilities:**
- Create an account and sign in — Organiser signs up with an email address and a protected sign-in before creating the household, and can recover access if the sign-in is forgotten [AUDIT-ADDED: 4 -- Security and Privacy Posture: the household holds children's allergy data, so account protection is an explicit capability, not an assumption]
- Create the household — Organiser names the household and becomes its first member
- Add member profiles — Organiser adds each person eating with them, including kid profiles with no login
- Set dietary rules per person — Organiser records allergies, religious rules, per-person vegetarian settings, and known dislikes; allergies are chosen from a standard allergen list, with the option to name a specific extra ingredient, so the safety check can match them reliably [AUDIT-ADDED: 1 -- per-feature depth walk: free-text allergies cannot be checked reliably, and the allergy rule is the product's hard safety promise (BRIEF.md, Constraints: Safety)]
- Confirm parental consent for kid profiles — When adding a kid profile, the organiser confirms they are the child's parent or guardian and sees exactly what is stored: a first name or nickname, an age band, and dietary rules [AUDIT-ADDED: 4 -- Compliance: children's data must be minimal and parent-controlled (BRIEF.md, Constraints: Privacy / children)]
- Set the weekly budget — Organiser states a rough weekly food budget
- Set the weekly schedule — Organiser marks which nights are short on time (e.g., 30-minute weeknights)
- Edit setup later — Organiser revisits and changes any of the above at any time

**Primary Flows & Alternates:**
- Happy path: organiser opens setup -> names the household -> adds members and their dietary rules -> sets budget and schedule -> setup complete, ready for the first plan
- Partial setup: organiser can save and return later; incomplete setup blocks plan generation only for the specific facts genuinely missing (e.g., no schedule set defaults to no time constraint, not a hard block)
- Later edit: organiser adds a new member or adjusts the budget or schedule after the household is already active — these changes apply to the next plan, not retroactively
- Mid-week allergy or religious-rule change: a new or tightened hard rule re-checks the current week immediately; any remaining dinner that now fails is flagged and the household is offered safe alternatives through One-Tap Meal Swap (FEAT-04), and the grocery list updates to match [MODIFIED: hard-rule changes now apply to the current week instead of waiting for the next plan, based on BRIEF.md's safety constraint that the app checks every suggestion before anyone sees it — a meal made unsafe by a new allergy must not stay on the plan]

**States:** Empty: a brand-new household shows a short, guided setup rather than a blank form. Loading: saving a setup step shows an inline confirmation, not a full-page spinner. Error: a failed save keeps the entered data on screen and offers a retry, never silently discards input. Offline-degraded: setup can be drafted offline and is held locally until connectivity returns to save.

**Validation & Limits:** Household name required (1–60 characters); at least one member (the organiser) required; each member's dietary rules are optional but an allergy, once entered, cannot be silently dropped without an explicit confirmation step; weekly budget must be a positive amount in the household's configured currency. Allergies are picked from a standard allergen list plus optional named ingredients; a kid profile stores no surname, birth date, photo, or contact detail — only a first name or nickname, an age band, and dietary rules; a household holds up to 12 member profiles, comfortably above the brief's 2–6 people. [AUDIT-ADDED: 1 -- per-feature depth walk: boundary values for allergens, kid-profile data, and member count were unstated]

**Access:** Maya (Organiser) has Full access to household setup, including kid profile data, per the Access Matrix in user-persona.md. Sam (Other Adult Member) has View access — he can see household facts and kid profiles' dietary rules but not change them. Jordan has no access in either form: as a young kid profile Jordan has no login, and the Later-phase older-kid login does not include setup. Riley (Operator, support) has View access only from v1, through Operator Read-Only Support Access (FEAT-22), and never sees kid profile data outside a specific safety report. An unauthorized visitor sees only a sign-in or invitation-acceptance screen, never household data. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** A one-time confirmation when setup is complete, telling the organiser what happens next: on the free tier, start picking this week's dinners (Manual Weekly Planning, FEAT-23); on the paid tier, the first AI plan is on its way. The organiser is also told when a mid-week rule change removes a meal from the current plan. [MODIFIED: synthesis check 3 — the draft promised "generate a first plan" to every household, which contradicted the free-tier default (BRIEF.md, Business Context: the free tier gets no AI)]

**Data Notes:** Captured: account sign-in details, household name, member list, per-member dietary rules, parental-consent confirmation for each kid profile, weekly budget, weekly schedule. Displayed: the current setup state to the organiser, including each dietary rule's change history (who changed it and when) and any open support requests. Derived: none. Source: organiser input only. [AUDIT-ADDED: 4 -- Audit Logging: a visible change history for allergy rules supports household trust and any safety investigation]

**Interactions:** Feeds every other feature; Household Invitations & Membership (FEAT-09) extends the member list it creates; Dietary Rules & Allergy Safety Engine (FEAT-02) reads the dietary rules captured here. Units, Currency & Locale Configuration (FEAT-16) is set during the same setup; Subscription & Billing Management (FEAT-14) starts every new household on the free tier; Account & Data Management (FEAT-18) removes members and deletes the household; a mid-week hard-rule change triggers re-checks through FEAT-02 and swaps through One-Tap Meal Swap (FEAT-04).

**Signals:** account_created, household_created, member_added, kid_profile_consent_confirmed, dietary_rule_added, dietary_rule_changed, budget_set, schedule_set, setup_completed, midweek_rule_change_recheck.

### Dietary Rules & Allergy Safety Engine

**ID:** FEAT-02

**Description:** Before any suggested meal reaches a household member, the app itself checks it against every household member's allergies and religious rules. This check runs independently of the AI — the AI proposes, the app verifies — so a single AI mistake can never reach the table.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief states this as a hard rule, not a preference: "the app itself checks every suggestion against the household's allergy list... the AI is never the last line of defence" (BRIEF.md, Vision, Constraints: Safety). This is the single most trust-critical feature in the product and must exist from day one. [RESEARCH-INFORMED: added market evidence that no profiled competitor (Mealime, AnyList, Samsung Food, Cozi) describes an app-enforced allergy check distinct from AI personalization or a manual exclusion filter (4 products, HIGH confidence), and a publicized supermarket AI meal-planner produced dangerous recipes — the failure mode this engine exists to prevent (news coverage, MEDIUM confidence)]

**Connected Entities:** Dietary Rule (read), Recipe (read), Planned Meal (update — attaches a "checked" badge)

**Key Capabilities:**
- Verify every suggestion — Every recipe considered for a household's plan is checked against that household's allergies and religious rules before it can appear, including recipes an adult picks by hand in Manual Weekly Planning (FEAT-23) and past weeks re-used from Weekly Plan History (FEAT-19) [MODIFIED: scope of the check extended to manual picks and re-used weeks so that no path onto the plan bypasses it, per BRIEF.md, Constraints: Safety]
- Show a safety badge — Every meal in the plan carries a "checked against allergies" badge with a standard "always check labels" disclaimer
- Block unsafe suggestions — A recipe that fails the check for any household member is removed from consideration for that household entirely, not just flagged
- Distinguish rule strength — Allergies and religious rules are hard filters; vegetarian settings apply per person with a shared-meal vegetarian option; dislikes are soft and never block a suggestion [RESEARCH-INFORMED: added the finding that existing products treat all restrictions as one undifferentiated exclusion list, and none layers hard rules against learned soft preferences (Feature Landscape, Absent Features)]
- Report a safety concern — Any adult member can flag a meal they believe is unsafe; it is removed from the household's plan at once, safe alternatives are offered, and the report reaches the operator for review [AUDIT-ADDED: 1 -- journey walk error handling: the draft gave households no way to act when they suspect a badge is wrong, which the brief's zero-incident goal (BRIEF.md, Success Criteria) requires]

**Primary Flows & Alternates:**
- Happy path: a candidate recipe is checked against all household members' hard rules before it is ever shown; a safe recipe displays its "checked against allergies" badge
- Hard-rule failure: a recipe that would violate any member's allergy or religious rule is silently excluded from that household's candidate pool — never shown, never suggested, never requiring the user to reject it themselves
- Vegetarian handling: a shared meal that isn't inherently vegetarian can carry a "vegetarian option" variant so one household with mixed diets can still share one plan
- Safety concern raised: an adult taps "report a safety concern" on a meal -> the meal leaves this household's plan immediately and the recipe is excluded for the household while the report is open -> the grocery list drops its ingredients -> the operator reviews the recipe's ingredient data and the household is told the outcome [AUDIT-ADDED: 1 -- counterpart symmetry: both the household (reporter) and the operator (reviewer) have a defined path]

**States:** Empty: N/A — this feature has no user-facing empty state; it runs invisibly behind every suggestion. Loading: the safety check completes before a suggestion is ever shown, so no separate loading state is visible to the user. Error: if a safety check cannot be completed for a candidate recipe, that recipe is excluded by default rather than shown unchecked — failing closed, never open. Offline-degraded: N/A — the safety check runs as part of plan generation, which requires connectivity; there is no offline mode for generating new suggestions.

**Validation & Limits:** Every household member's allergy and religious-rule data must be checked against every ingredient of every candidate recipe with no partial matching shortcuts; a recipe missing complete ingredient data cannot pass the check and is excluded rather than assumed safe. A safety-concern report needs the meal and an optional short note (up to 500 characters); a recipe under an open report is never re-offered to that household until the report is resolved.

**Access:** The safety badge is visible to everyone who can see a meal — Maya, Sam, and the Later-phase older-kid login — per the Access Matrix in user-persona.md; only Maya can change the dietary rules the engine reads (Household Setup column). Maya (Full) and Sam (Own-only) can raise safety concerns (Safety Reports column); Riley (Operator, support) has View access to reports to investigate them. Jordan as a young kid profile has no login and is protected through the rules Maya sets. An unauthorized visitor sees nothing, since this engine has no standalone screen. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** A safety-concern report sends the reporter an acknowledgement, tells the organiser a meal was removed from the plan, and sends the report to the operator by transactional email; the household is told when the review is resolved. Otherwise the engine runs silently within plan generation, manual picks, and swaps, and its badges are visible wherever a meal is shown. [AUDIT-ADDED: 1 -- communications required by the new safety-concern capability]

**Data Notes:** Captured: safety-concern reports (meal, reporting member, optional note) as Support Requests. Displayed: the "checked against allergies" badge and "always check labels" disclaimer on every meal, and a plain reason on any recipe marked ineligible. Derived: the pass/fail safety determination for each candidate recipe against each household's Dietary Rules. Source: Dietary Rule data from Household Setup (FEAT-01) and ingredient data from each Recipe.

**Interactions:** Runs within AI Weekly Dinner Plan Generation (FEAT-03), Manual Weekly Planning (FEAT-23), One-Tap Meal Swap (FEAT-04), Older-Kid Dinner Voting (FEAT-17), and re-use in Weekly Plan History (FEAT-19); reads data from Household Setup & Member Profiles (FEAT-01); its badge is displayed wherever Recipe Library (FEAT-08) or Recipe Import (FEAT-10) content is shown; safety-concern reports are reviewed through Operator Read-Only Support Access (FEAT-22).

**Signals:** safety_check_run, safety_check_failed (recipe excluded), safety_badge_shown, safety_check_data_incomplete, safety_concern_reported, safety_concern_resolved.

### AI Weekly Dinner Plan Generation

**ID:** FEAT-03

**Description:** Every week, the household receives a proposed 7-day dinner plan that fits everyone's allergies, diets, dislikes, schedule, and budget, and makes use of what the household says is already in the fridge.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** This is the product's central promise: "AI proposes a realistic 7-day dinner plan every week that respects all of it" (BRIEF.md, Vision). It is the paid tier's primary value (BRIEF.md, Business Context) and the feature every other Core feature exists to support. [RESEARCH-INFORMED: added the pricing finding that where AI-generated planning exists (Mealime, Samsung Food) it sits on the paid tier while list and recipe management stay free (vendor pricing pages, MEDIUM confidence), matching the brief's tier split]

**Connected Entities:** Weekly Plan (create), Planned Meal (create), Recipe (read), Dietary Rule (read), Pantry Item (read), Rating (read), Subscription (read — gates access)

**Key Capabilities:**
- Generate the week's plan — Household receives seven dinners that respect every hard dietary rule, the stated schedule, and the budget
- Use up the pantry — The plan favors recipes that use ingredients the household has already logged as on hand
- Show cost and time per meal — Each dinner shows its rough cost and cook time, and the week shows its estimated total against the household's budget [AUDIT-ADDED: 1 -- per-feature depth walk: the brief's budget constraint needs a visible check, not only a hidden input]
- Scale to the household — Ingredient quantities are sized for the number of people eating, so the grocery list buys the right amount [RESEARCH-INFORMED: added competitor context — Mealime is praised for recipes that are easy to adjust for family size (App Store reviews, HIGH confidence) and Samsung Food is criticised when serving changes do not carry through to the list (aggregated reviews, HIGH confidence)]
- Approve the week's plan — The organiser reviews the proposal, including any swap suggestions from other adults, and approves it in one tap [AUDIT-ADDED: 1 -- journey walk: BRIEF.md, Target Users & Roles states the organiser "approve[s] the weekly plan"; the draft had no approval step]
- Learn from ratings over time — Later plans favor meals the household has rated highly and avoid ones rated poorly

**Primary Flows & Alternates:**
- Happy path: plan generates automatically once a week -> household opens it to see seven dinners, each with cook time, rough cost, and dietary badges, and at least one flagged as using existing pantry items
- Insufficient data: a brand-new household with minimal setup (e.g., no pantry items logged) still receives a complete, safe plan — pantry-awareness is a refinement, not a precondition for getting a plan
- Free tier: households on the free tier do not receive an AI-generated plan and instead start the week in Manual Weekly Planning (FEAT-23), with a clear, unobtrusive option to upgrade [MODIFIED: now points to the Manual Weekly Planning feature added in synthesis, which delivers the brief's free-tier "manual planning"]
- Budget cannot be met: when no safe week fits the budget, the household gets the closest-fitting plan with a plain note showing the estimated overrun, never a silent overspend [AUDIT-ADDED: 1 -- error handling for the budget constraint]
- Not approved in time: a plan the organiser has not approved by the start of the week is adopted as proposed, so the household is never without a plan [AUDIT-ADDED: 1 -- reversal path for the approval step]

**States:** Empty: a paid household that has not yet had a plan generated sees a clear "your first plan is on its way" message rather than a blank week. Loading: plan generation shows a short, explained wait (typically well under a minute) rather than an indefinite spinner. Error: if generation fails, the household keeps the previous week's plan visible and is offered a retry, never left with no plan at all. Offline-degraded: an already-generated plan remains fully viewable offline; generating a new plan requires connectivity.

**Validation & Limits:** Generation requires at least one household member with complete dietary-rule data (even "no restrictions" is an explicit statement) and a schedule; a plan always proposes exactly seven dinners, one per day, regardless of household size. Only the organiser approves; approval can be given once per week and later changes happen through swaps; the weekly estimated total is shown in the household's currency.

**Access:** Maya (Organiser) has Full access to the Weekly Plan, including approving it, per the Access Matrix in user-persona.md; Sam (Other Adult Member) has View access and shapes the plan through swap suggestions (FEAT-04). Jordan has no direct access as a young kid profile without a login; the Later-phase older-kid login has View access. Riley (Operator, support) has View access from v1 through FEAT-22. Free-tier households see the upgrade option in place of AI generation. An unauthorized visitor sees no plan content. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** Triggers Weekly Plan Ready Notification (FEAT-07) once generation completes.

**Data Notes:** Captured: the organiser's approval (generation itself is triggered automatically on a schedule). Displayed: seven Planned Meals with cook time, rough cost, safety/dietary badges, and the week's estimated total against budget. Derived: the meal selection itself, computed from Recipe data filtered by Dietary Rules & Allergy Safety Engine (FEAT-02), weighted by Pantry Items and past Ratings, sized for the household, and fit to the household's budget and schedule. Source: household setup data, recipe data, pantry data, and rating history.

**Interactions:** Depends on Household Setup & Member Profiles (FEAT-01) for constraints, Dietary Rules & Allergy Safety Engine (FEAT-02) for safety filtering, Pantry-Aware Suggestions (FEAT-05) for pantry weighting, Meal Rating & Preference Learning (FEAT-12) for taste learning, Recipe Library (FEAT-08) and Recipe Import (FEAT-10) for candidate recipes, and Subscription & Billing Management (FEAT-14) for tier gating. Feeds Weekly Plan Ready Notification (FEAT-07), Shared Grocery List (FEAT-06), One-Tap Meal Swap (FEAT-04), Leftover Rollover to Lunches (FEAT-11), and Older-Kid Dinner Voting (FEAT-17). Also reads Units, Currency & Locale Configuration (FEAT-16) for cost display; free-tier households are routed to Manual Weekly Planning (FEAT-23).

**Signals:** plan_generation_started, plan_generation_completed, plan_generation_failed, plan_viewed, plan_approved, plan_auto_adopted, plan_over_budget, plan_used_pantry_item (count).

### Manual Weekly Planning

**ID:** FEAT-23

**Description:** Any household, on either tier, can build the week's dinners by hand — picking a recipe for each night from the library — and the shared grocery list builds itself from those picks. This is the free tier's planning experience, and the way a paid household hand-picks any night it prefers to choose itself.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Business Context states "the free tier covers manual planning and the shared grocery list," yet the draft had no feature that delivers manual planning — free households could only see an upgrade prompt. Market research shows weekly plan-to-shopping-list automation in all four profiled products (Feature Landscape, Common Features). Core because every new household starts on the free tier and plans through this feature until it upgrades, so both the free product and the path to paid depend on it; MVP because the free tier exists at launch. [AUDIT-ADDED: 2 -- Core: plan-to-list weekly planning is a Common Feature present in all 4 profiled competitors (vendor feature pages, HIGH confidence) and the brief names manual planning as the free tier's core offer; without it the default tier has no way to plan a week]

**Connected Entities:** Weekly Plan (create, update), Planned Meal (create, update, delete), Recipe (read), Dietary Rule (read), Swap Suggestion (create — for other adult members)

**Key Capabilities:**
- Pick a dinner for a night — Organiser chooses a recipe from the starter library or the household's imported recipes for any night of the week
- See only safe choices — Recipes that break a household member's allergy or religious rule are marked ineligible with a plain reason and cannot be placed
- Change or clear a night — Organiser replaces or empties any night's pick
- Suggest a pick — Other adult members suggest a dinner for a night, which the organiser accepts or declines
- Build the list automatically — Every pick adds its ingredients to the shared grocery list, and every change updates it

**Primary Flows & Alternates:**
- Happy path: organiser opens next week -> taps a night -> browses or searches recipes -> picks one -> repeats for the nights they want -> the grocery list fills in as they go
- Partial week: nights left empty are allowed; the list reflects only the planned nights, and empty nights show a gentle "nothing planned" marker rather than an error
- Unsafe pick attempted: a recipe that fails the hard-rule check shows which member's rule it breaks and cannot be placed
- Paid household picking by hand: a paid household can replace any AI-proposed night with a hand-picked recipe through the same safe-choice list

**States:** Empty: a brand-new week shows seven empty nights with a prompt to pick the first dinner. Loading: recipe choices appear within about a second with an inline indicator. Error: a failed save keeps the pick on screen with a retry, never silently dropping it. Offline-degraded: the current week remains fully viewable offline; placing a new pick requires connectivity so every pick passes the safety check before it lands on the plan.

**Validation & Limits:** At most one dinner per night and seven nights per week; a week can be planned up to one week ahead; every pick must pass the same allergy and religious-rule check as AI suggestions; one open pick suggestion per member per night.

**Access:** Maya (Organiser) has Full access per the Manual Planning column of the Access Matrix in user-persona.md; Sam (Other Adult Member) has Own-only access — he suggests picks that Maya accepts or declines; Jordan (young kid profile, no login) and the Later-phase older-kid login have none; Riley (Operator, support) has none. An unauthorized visitor cannot see or change the week.

**Communications:** A suggested pick notifies the organiser, and the suggester is told when it is accepted or declined; no other notifications.

**Data Notes:** Captured: the organiser's per-night recipe picks and other members' pick suggestions. Displayed: the week with each night's recipe, cook time, rough cost, safety badge, and the week's estimated total against budget. Derived: the grocery list contents and the weekly cost estimate. Source: household input plus Recipe Library (FEAT-08) and Recipe Import (FEAT-10) data.

**Interactions:** Depends on Recipe Library (FEAT-08), Recipe Import from Web Link (FEAT-10), Dietary Rules & Allergy Safety Engine (FEAT-02), and Household Setup & Member Profiles (FEAT-01) for budget and schedule display; feeds Shared Grocery List (FEAT-06), Tonight's Dinner Reminder (FEAT-13), and Meal Rating & Preference Learning (FEAT-12); Subscription & Billing Management (FEAT-14) routes free and downgraded households here.

**Signals:** manual_week_started, manual_meal_picked, manual_pick_blocked_unsafe, manual_pick_suggested, manual_week_completed.

### One-Tap Meal Swap

**ID:** FEAT-04

**Description:** Any dinner in the plan can be replaced with a different one in a single tap, and the grocery list updates immediately to match. The organiser swaps directly; other adult members suggest a swap, which the organiser accepts or declines with one tap. [MODIFIED: other adult members now suggest rather than make swaps, based on BRIEF.md, Target Users & Roles — other adults "see the plan and suggest swaps" while the organiser approves the plan (founder-stated role split)]

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief calls this out explicitly: "Any meal can be swapped with one tap, and the grocery list updates instantly" (BRIEF.md, Vision, The Experience). Plans that cannot flex to real life are abandoned; this keeps the plan usable when reality changes. [RESEARCH-INFORMED: added the finding that multi-restriction households see sharply narrowed options in preference-filtered planners (Mealime App Store reviews, MEDIUM confidence), which is why the limited-alternatives explanation below matters]

**Connected Entities:** Planned Meal (update), Weekly Plan (update), Swap Suggestion (create, update), Grocery List (update — indirectly, via FEAT-06)

**Key Capabilities:**
- Swap a meal — Household member replaces one night's dinner with an alternative in one tap
- See safe alternatives only — Every offered alternative has already passed the allergy safety check
- Instant list update — The grocery list adjusts automatically to reflect the swap
- Suggest a swap — An other adult member picks a safe alternative and sends it to the organiser as a suggestion [MODIFIED: added per BRIEF.md's role split, above]
- Review suggestions — The organiser sees pending suggestions on the plan and accepts or declines each with one tap [MODIFIED: added per BRIEF.md's role split, above]

**Primary Flows & Alternates:**
- Happy path: user taps swap on a planned meal -> sees a short list of safe alternatives -> picks one -> the plan and grocery list update immediately
- No good alternative available: if the safety and schedule constraints leave very few options, the app still shows what qualifies rather than an empty list, and explains briefly why choices are limited
- Repeated swap: a meal can be swapped more than once in the same week without restriction
- Suggested swap: Sam taps swap on Friday's dinner -> picks a safe alternative -> Maya is told a suggestion is waiting -> she accepts with one tap and the plan and list update, or declines and Sam is told the original stays
- Night passes: a suggestion not answered before its night lapses quietly, and the suggester sees that it lapsed [AUDIT-ADDED: 1 -- counterpart symmetry: the suggester and the organiser each see the outcome]

**States:** Empty: N/A — a meal to swap always exists once a plan is generated. Loading: alternatives appear within a couple of seconds with a brief inline indicator. Error: a failed swap leaves the original meal in place rather than an empty slot, with a retry option. Offline-degraded: swapping requires connectivity to fetch safe alternatives; while offline, the current plan remains viewable but swap is disabled with a clear explanation.

**Validation & Limits:** An alternative must pass the same allergy/religious hard-rule check as original generation; no more than one active swap operation per meal slot at a time (prevents duplicate swaps from a double tap). One open suggestion per member per meal slot; a suggestion lapses when its night passes.

**Access:** Maya (Organiser) has Full access to Meal Swap per the Access Matrix in user-persona.md and swaps directly. Sam (Other Adult Member) has Own-only access: he raises his own swap suggestions, which Maya accepts or declines. Jordan (young kid profile) and the Later-phase older-kid login have no swap access — older kids influence dinners through Older-Kid Dinner Voting (FEAT-17). Riley (Operator, support) has none. An unauthorized visitor cannot swap or suggest anything. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** A swap suggestion notifies the organiser; the suggesting member is told when it is accepted, declined, or lapses. A same-day swap also triggers the correction nudge in Tonight's Dinner Reminder (FEAT-13). A direct swap by the organiser sends no separate notification. [MODIFIED: suggestion notifications added with the suggest-and-approve role split]

**Data Notes:** Captured: swap suggestions (member, night, proposed recipe, outcome). Displayed: the current meal, its safe alternatives, and any pending suggestions. Derived: the alternatives list, computed the same way as original plan generation but scoped to one slot. Source: Recipe data filtered by Dietary Rules & Allergy Safety Engine (FEAT-02).

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) or Manual Weekly Planning (FEAT-23) for the plan it modifies and Dietary Rules & Allergy Safety Engine (FEAT-02) for safe alternatives; updates feed Shared Grocery List (FEAT-06) automatically and trigger corrections in Tonight's Dinner Reminder (FEAT-13); a mid-week rule change in Household Setup (FEAT-01) opens a swap for any meal that now fails.

**Signals:** meal_swap_opened, meal_swap_completed, meal_swap_failed, meal_swap_alternatives_limited, swap_suggested, swap_suggestion_accepted, swap_suggestion_declined, swap_suggestion_lapsed.

### Pantry-Aware Suggestions

**ID:** FEAT-05

**Description:** The household tells Plateful what it already has on hand, and the weekly plan favors recipes that use those ingredients up before they go to waste.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief's central "nothing gets thrown away" promise depends on this (BRIEF.md, Vision, The Experience: "Thursday says 'uses the spinach and feta you already have'"). The brief leaves pantry depth as an open question and leans toward a simple "tell me what I have" model rather than detailed inventory tracking. [RESEARCH-INFORMED: added the finding that the only pantry-aware competitor (Samsung Food, paid tier) is described as "basic" and unable to reflect leftovers or real kitchen behavior (independent review, MEDIUM confidence), so pantry awareness is a documented gap Plateful targets alongside Leftover Rollover to Lunches (FEAT-11)]

**Connected Entities:** Pantry Item (create, update, delete)

**Key Capabilities:**
- Log what's on hand — Household member notes an ingredient they already have, in plain language
- See it used in the plan — A plan that uses a logged pantry item calls that out explicitly (e.g., "uses the spinach and feta you already have")
- Clear used items — Household can mark a pantry item as used up once the week's plan consumes it; after the dinner that used an item has passed, a one-tap "used it up?" prompt appears on the pantry list [AUDIT-ADDED: 3 -- entity coverage: the Pantry Item's Used state had no trigger]
- Keep it off the list — A logged pantry item is left off the week's grocery list, so the household does not buy what it already has

**Primary Flows & Alternates:**
- Happy path: household member adds "spinach, feta" before the week's plan generates -> plan uses them in a dinner and calls it out
- Nothing logged: a household that logs no pantry items still receives a complete plan; pantry-awareness simply has nothing to weight toward that week
- Stale item: a logged item that goes unused for several weeks is not automatically removed — the household clears it manually when it is used or gone

**States:** Empty: an empty pantry list shows a short prompt to add what's on hand, not a bare screen. Loading: adding an item confirms instantly. Error: a failed save keeps the typed item visible with a retry option. Offline-degraded: pantry items can be added offline and sync once connectivity returns.

**Validation & Limits:** Item name required (1–80 characters, free text — no rigid inventory schema); no fixed limit on how many items a household can log, though the feature is designed for a short, current list rather than a full inventory.

**Access:** Maya and Sam have Full access to Pantry Input per the Access Matrix in user-persona.md. Logging pantry items (and keeping them off the grocery list) works on both tiers; the plan choosing dinners to use them up — pantry-aware suggestions — requires the paid tier (BRIEF.md, Business Context). Jordan (young kid profile) and the Later-phase older-kid login have none. Riley (Operator, support) has View access from v1 through FEAT-22. An unauthorized visitor sees no pantry data. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** N/A — this feature has no notifications of its own; its effect surfaces as a callout inside the weekly plan.

**Data Notes:** Captured: free-text pantry item names, added by household members. Displayed: the current pantry list and, within the plan, which dinners use a logged item. Derived: none. Source: household input only.

**Interactions:** Feeds AI Weekly Dinner Plan Generation (FEAT-03), which reads current Pantry Items when selecting meals on the paid tier, and Shared Grocery List (FEAT-06), which leaves logged items off the list on either tier; gated for plan weighting by Subscription & Billing Management (FEAT-14).

**Signals:** pantry_item_added, pantry_item_cleared, pantry_item_used_in_plan. pantry_used_prompt_answered.

### Shared Grocery List

**ID:** FEAT-06

**Description:** One combined grocery list, generated from the week's plan and grouped by supermarket aisle, that the whole household sees and ticks off together in real time — including in the store with a weak signal.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief describes this as one of the two concrete deliverables of the product (alongside the plan itself) and states it "must feel instant and must keep working in a supermarket with bad signal" (BRIEF.md, Vision, Scale & Non-Functional Expectations). It directly replaces the group-chat list called out in the Problem Statement. [RESEARCH-INFORMED: added the finding that a real-time shared household list is the most-praised feature wherever it is done well (AnyList, Cozi reviews, HIGH confidence), while the one competitor combining AI plans with household sharing reports members seeing the plan but not the matching list (Samsung Food, MEDIUM confidence) — so every member must always see the same plan and the same list]

**Connected Entities:** Grocery List (create, update), Grocery List Item (create, update, delete)

**Key Capabilities:**
- Auto-generate from the plan — The list is built from the ingredients of the week's plan, grouped by aisle
- Add manually — Any household member can add an item the plan didn't include
- Tick off live — Ticking an item is visible to every household member immediately
- Work offline in the store — Changes made with no signal sync automatically once connectivity returns
- Combine repeated ingredients — The same ingredient needed by several dinners appears once, with the combined quantity [RESEARCH-INFORMED: added competitor context — automatic consolidation across recipes is praised as a time-saver (Samsung Food and AnyList reviews, MEDIUM confidence)]
- Edit or remove an item — Any member with list access corrects a quantity or removes an item [AUDIT-ADDED: 3 -- entity coverage: Grocery List Item had no edit path]
- "Already have it" — A member marks a plan-derived item as already at home; it leaves the list and can be added to the pantry in the same tap [AUDIT-ADDED: 1 -- journey walk: shoppers find items at home that were never logged]

**Primary Flows & Alternates:**
- Happy path: plan generates -> list builds automatically, grouped by aisle -> household member adds "more yoghurt" from home -> partner ticks items off in the store, seeing the same live-updating list
- Offline edit: a household member ticks off items with no signal; the ticks are held locally and sync the moment connectivity returns, without creating duplicates or conflicts
- Swap-driven update: when a meal is swapped (FEAT-04), the list updates its ingredients automatically without the user having to re-check anything
- Week rollover: when the week's list is archived, unticked manually added items carry into the next week's list so nothing the household still needs is lost [AUDIT-ADDED: 3 -- entity coverage: Grocery List archive behavior]

**States:** Empty: before a plan exists, the list shows a short explanation that it will populate once the first plan is generated, plus the option to add items manually in the meantime. Loading: newly generated list items appear within a couple of seconds with an inline indicator. Error: a failed tick or add retries automatically in the background and only surfaces an error if it cannot eventually succeed. Offline-degraded: full read and tick/add functionality remains available offline; changes queue and sync when connectivity returns.

**Validation & Limits:** Manually added item name required (1–80 characters); aisle grouping uses the household's configured aisle names; duplicate manual entries for the same ingredient are merged rather than shown twice.

**Access:** Maya and Sam have Full access to the Grocery List per the Access Matrix in user-persona.md; the Later-phase older-kid login has Full access to add and tick items (the brief's teenager adding "more yoghurt"), but not to household-level list settings such as aisle names, which belong to Household Setup (FEAT-16). Jordan as a young kid profile has no login. Riley (Operator, support) has View access from v1 through FEAT-22. An unauthorized visitor cannot see the list. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** N/A — updates are shown live in the list itself rather than pushed as separate notifications.

**Data Notes:** Captured: manually added items (with which member added them), quantity edits, removals, "already have it" marks, and tick state. Displayed: the combined, aisle-grouped list, with a small "added by" label on manual items. Derived: the auto-generated portion of the list, computed from the week's Planned Meals' ingredients (combined across dinners and sized for the household) minus logged Pantry Items. Source: AI Weekly Dinner Plan Generation (FEAT-03) or Manual Weekly Planning (FEAT-23) output, Pantry-Aware Suggestions (FEAT-05) data, and direct household input.

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03), Manual Weekly Planning (FEAT-23), and One-Tap Meal Swap (FEAT-04) for its auto-generated contents, Pantry-Aware Suggestions (FEAT-05) to avoid listing items already on hand, and Units, Currency & Locale Configuration (FEAT-16) for aisle names and units; read by Weekly Plan History (FEAT-19) and, in a later phase, Online Grocery Ordering Handoff (FEAT-20).

**Signals:** grocery_list_generated, grocery_item_added_manually, grocery_item_ticked, grocery_item_edited, grocery_item_removed, grocery_item_already_have, grocery_items_combined, grocery_item_synced_after_offline, grocery_items_carried_over.

### Weekly Plan Ready Notification

**ID:** FEAT-07

**Description:** The household is told, on a predictable schedule, that next week's plan is ready to review.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief's own description of the core experience begins here: "It's Sunday evening and a notification arrives: 'Next week's plan is ready.'" (BRIEF.md, The Experience). Without this, the weekly plan is invisible until someone happens to check. [INFERRED: carried from Visionary draft]

**Connected Entities:** Weekly Plan (read)

**Key Capabilities:**
- Notify when the plan is ready — Household members with notifications enabled are told as soon as generation completes
- Control who gets notified — Each household member can enable or disable this notification for themselves
- Choose when the plan arrives — The organiser picks the day and rough time the weekly plan arrives (Sunday evening by default) [AUDIT-ADDED: 4 -- Settings and Preferences: the brief's Sunday-evening moment should be the default, not a fixed rule every household must live with]

**Primary Flows & Alternates:**
- Happy path: plan generation completes -> enabled household members receive a "next week's plan is ready" notification -> tapping it opens the plan directly
- Notification disabled: a household member who has turned this off simply sees the plan the next time they open the app, with no notification sent
- Delayed delivery: if notification delivery itself fails or is delayed, the plan is still available in-app immediately upon generation, unaffected by the notification path

**States:** Empty: N/A — this feature has no content of its own beyond the notification text. Loading: N/A — delivery is near-instant once generation completes. Error: a failed notification delivery does not block or delay the plan's in-app availability. Offline-degraded: a notification sent while a device is offline is delivered once the device reconnects, per standard device-notification behavior.

**Validation & Limits:** At most one "plan ready" notification per household per week, tied to generation completion, not user action. The plan-arrival day can be any day of the week; the time is chosen from a small set of evening and morning slots.

**Access:** Maya and Sam each control their own notification preference per the Access Matrix in user-persona.md (Notification Prefs column: Full for Maya, who also sets when the plan arrives; Own-only for Sam). Neither kid row has notification settings — young kid profiles have no login, and the Later-phase older-kid login does not receive plan notifications. Riley (Operator, support) has none. An unauthorized visitor receives nothing. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** This feature is itself the communication: a device notification, "Next week's plan is ready." Where a member has not enabled device notifications, the same message goes by transactional email instead, unless they have turned it off. [AUDIT-ADDED: 4 -- External-Service Integrations: device notifications from a web app are not available on every phone, so an email fallback keeps the Sunday rhythm the brief describes]

**Data Notes:** Displayed: N/A — this is a notification, not a data view. Derived: none. Source: the completion event from AI Weekly Dinner Plan Generation (FEAT-03).

**Interactions:** Triggered by AI Weekly Dinner Plan Generation (FEAT-03); its preference toggle lives within Household Setup & Member Profiles (FEAT-01).

**Signals:** plan_ready_notification_sent, plan_ready_notification_opened, plan_ready_notification_disabled. plan_arrival_day_changed, plan_ready_email_sent.

### Recipe Library (Starter Recipes)

**ID:** FEAT-08

**Description:** A built-in library of starter recipes the AI plan draws from and that household members can browse directly.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief names "a starter recipe library" explicitly as part of the recipe sources the product needs (BRIEF.md, Ecosystem & Integrations). Without a starter library, plan generation would have nothing to propose from day one, before any household has imported recipes of their own. [INFERRED: carried from Visionary draft]

**Connected Entities:** Recipe (create — starter content, read)

**Key Capabilities:**
- Browse recipes — Household member looks through the library directly, outside the weekly plan
- View recipe detail — Household member sees ingredients, steps, cook time, rough cost, and dietary badges for one recipe
- Search and filter — Household member finds recipes by name, ingredient, or dietary badge

**Primary Flows & Alternates:**
- Happy path: household member opens the library -> searches or filters -> opens a recipe's detail view with ingredients, steps, and badges
- No results: a search with no matches shows a plain "nothing found" message and suggests broadening the search, never an empty white screen
- Cross-reference from plan: opening a recipe from within the weekly plan shows the same detail view as browsing the library directly
- Ineligible recipe: a recipe that breaks a household member's allergy or religious rule, or whose ingredient data cannot be fully verified, is shown with a plain explanation of why it is not eligible for this household and cannot be added to the plan [MODIFIED: synthesis check 14 — the Allergy-Safe Swap Recovery journey relies on the library explaining ineligibility, which no feature specified]

**States:** Empty: N/A — the starter library ships pre-populated and is never empty for any household. Loading: search results appear within about a second. Error: a failed detail-view load offers a retry without losing the search context. Offline-degraded: previously viewed recipes remain available offline; new searches require connectivity.

**Validation & Limits:** Search query has no minimum length; results are capped to a manageable page size with further results loaded on scroll.

**Access:** Maya and Sam have Full access to the Recipe Library per the Access Matrix in user-persona.md (browsing here, importing through FEAT-10); the Later-phase older-kid login has View access. Jordan as a young kid profile has no login. Riley (Operator, support) has View access from v1 through FEAT-22. An unauthorized visitor cannot browse the library. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** N/A — browsing is a self-initiated, in-app action with no notifications.

**Data Notes:** Displayed: recipe name, ingredients, steps, cook time, rough cost, dietary badges. Derived: dietary badges, computed by Dietary Rules & Allergy Safety Engine (FEAT-02) against the household's rules when viewed. Source: starter content plus recipes added via Recipe Import from Web Link (FEAT-10).

**Interactions:** Feeds AI Weekly Dinner Plan Generation (FEAT-03) as its candidate pool alongside Recipe Import from Web Link (FEAT-10); its badges depend on Dietary Rules & Allergy Safety Engine (FEAT-02). It is also the pick list for Manual Weekly Planning (FEAT-23).

**Signals:** recipe_library_opened, recipe_searched, recipe_detail_viewed, recipe_search_empty. recipe_ineligible_explained.

### Household Invitations & Membership

**ID:** FEAT-09

**Description:** The organiser invites other adults to join the household, so everyone sees the same plan and the same grocery list.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief states plainly that "everyone in a household sees the same plan and grocery list" (BRIEF.md, Target Users & Roles) and that growth is expected to come from "households inviting other households" (BRIEF.md, Business Context) — membership within a household is the mechanism that makes shared use possible at all. [MODIFIED: growth between households is now owned by Invite Another Household (FEAT-24); this feature covers membership within one household only, so the two are no longer conflated]

**Connected Entities:** Invitation (create, update), Member Profile (create — on acceptance)

**Key Capabilities:**
- Send an invitation — Organiser invites another adult by a simple, shareable method
- Accept an invitation — Invited adult joins the household and gains their role's access
- Manage outstanding invitations — Organiser can see and revoke a pending invitation
- Hand over the organiser role — The organiser makes another adult member the organiser, for example before stepping back from planning [AUDIT-ADDED: 3 -- relationship coverage: the Household–organiser relationship had no way to change hands, leaving a household stranded if the organiser leaves]
- Leave the household — An other adult member leaves on their own; their ratings stay only as anonymous influence on future plans and their profile is removed [AUDIT-ADDED: 3 -- entity coverage: Member Profile's removal could only be done by the organiser]

**Primary Flows & Alternates:**
- Happy path: organiser sends an invitation -> invited adult accepts -> a new Member Profile is created for them with Other Adult Member access
- Expired invitation: an invitation not accepted within a reasonable window expires automatically and can be resent
- Revoked invitation: organiser revokes a pending invitation before it is accepted; the invited person sees a clear "this invitation is no longer valid" message if they try to use it afterward
- Organiser leaving: an organiser who wants to leave must first hand over the role to another adult, or delete the household through Account & Data Management (FEAT-18)

**States:** Empty: a household with no outstanding invitations shows a plain "invite someone" prompt rather than an empty list with no explanation. Loading: sending an invitation confirms within a couple of seconds. Error: a failed invitation send preserves the entered contact detail and offers a retry. Offline-degraded: composing an invitation requires connectivity to send; a drafted invitation is held locally until it can be sent.

**Validation & Limits:** A contact detail is required to send an invitation; a household may have multiple outstanding invitations at once; an already-active member cannot be re-invited. Invitations expire after 14 days; the organiser role can only pass to an active adult member, who must accept it.

**Access:** Only Maya (Organiser) can send or revoke invitations and hand over the organiser role (Household Invitations column: Full for Maya, None for Sam) per the Access Matrix in user-persona.md. Sam can leave the household himself (Account & Data column, Own-only). Neither kid row can invite or be invited in MVP; an older-kid login, if introduced Later, would be created by the organiser. Riley (Operator, support) has none. An accepted invitee gains Other Adult Member access automatically. An unauthorized visitor can only act on a specific invitation link addressed to them. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** Sends the invitation itself (e.g., a shareable link or message) and a confirmation to the organiser once it is accepted. The organiser is also told when a member leaves, and the new organiser is asked to accept a hand-over.

**Data Notes:** Captured: the invited person's contact detail and the invitation's state. Displayed: outstanding and accepted invitations to the organiser. Derived: none. Source: organiser input.

**Interactions:** Extends the Member Profile list created by Household Setup & Member Profiles (FEAT-01); feeds Member Onboarding (FEAT-15) for the accepted invitee's first-use experience.

**Signals:** invitation_sent, invitation_accepted, invitation_revoked, invitation_expired. organiser_role_transferred, member_left_household.

## Important Features

### Recipe Import from Web Link

**ID:** FEAT-10

**Description:** A household member can save a recipe from any website by pasting its link, adding it to their own recipe pool alongside the starter library.

**Priority:** Important

**Phase:** v1

**Type:** User-Facing

**Rationale:** The brief names this directly: "saving recipes from any website by pasting a link" (BRIEF.md, Ecosystem & Integrations), while also flagging that "whether importing recipes from other sites is legal is an open question" (BRIEF.md, Open Questions). Phased to v1 rather than MVP because the starter library (FEAT-08) already gives new households a working candidate pool on day one; import extends personalization once the core loop is proven and the legal question is resolved. [RESEARCH-INFORMED: added the finding that recipe import is a common feature (AnyList, Samsung Food), that reviewers report import reliability regressing after an upgrade (AnyList, MEDIUM confidence) — so review-before-save and manual fallback are essential — and that capping free imports at five for life is flagged as a hard limit (AnyList, MEDIUM confidence), so imports are not capped by tier]

**Connected Entities:** Recipe (create — imported)

**Key Capabilities:**
- Import by link — Household member pastes a web link and the recipe's ingredients, steps, and cook time are extracted into the household's recipe pool
- Review before saving — Household member confirms or edits extracted details before the recipe is saved
- See imported recipes alongside starter ones — Imported recipes appear in the same library and are eligible for the weekly plan
- Edit or remove an imported recipe — Household member corrects an imported recipe's details later, or removes it from the household's pool; the change is re-checked for safety before it can appear in a plan again [AUDIT-ADDED: 3 -- entity coverage: imported Recipes had no edit or delete path]

**Primary Flows & Alternates:**
- Happy path: household member pastes a link -> the recipe's details are extracted -> member reviews and confirms -> it is saved to the household's recipe pool and can appear in future plans
- Extraction failure: a link that cannot be parsed prompts the member to enter the recipe details manually instead of failing silently
- Duplicate import: importing a link already saved for the household surfaces the existing recipe rather than creating a duplicate

**States:** Empty: N/A — this feature has no standing list of its own; imported recipes appear inside Recipe Library (FEAT-08). Loading: extraction shows a brief, explained progress indicator (typically a few seconds). Error: a failed extraction offers manual entry as a fallback rather than a dead end. Offline-degraded: importing requires connectivity to fetch and parse the link; a pasted link is held as a draft if offline and processed once reconnected.

**Validation & Limits:** A well-formed web address is required; extracted ingredient and step text is capped to a reasonable length consistent with other recipes in the library. No lifetime cap on imports on either tier; a household can import up to 30 recipes a week to guard against misuse; an imported recipe must have at least one ingredient to be saved, and one with an unrecognizable ingredient cannot pass the safety check until the ingredient is clarified.

**Access:** Maya and Sam have Full access to import, edit, and remove imported recipes per the Recipe Library column of the Access Matrix in user-persona.md; the Later-phase older-kid login has View access only and cannot import. Jordan as a young kid profile has no login. Riley (Operator, support) has View access from v1 through FEAT-22. An unauthorized visitor cannot import. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** N/A — import is a self-initiated, in-app action with no notifications.

**Data Notes:** Captured: the source link and the member's confirmed/edited recipe details. Displayed: the imported recipe within the library. Derived: the initial extraction (ingredients, steps, cook time) from the linked page, subject to member review before saving. Source: the external web page plus member confirmation.

**Interactions:** Feeds Recipe Library (FEAT-08) and, through it, AI Weekly Dinner Plan Generation (FEAT-03); its badges depend on Dietary Rules & Allergy Safety Engine (FEAT-02) once saved.

**Signals:** recipe_import_started, recipe_import_succeeded, recipe_import_failed, recipe_import_manual_fallback. imported_recipe_edited, imported_recipe_removed.

### Leftover Rollover to Lunches

**ID:** FEAT-11

**Description:** Leftovers from a planned dinner are carried forward as a suggested lunch on a following day, so extra portions get eaten instead of thrown away.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief names this directly as part of the core vision: "Leftovers roll into lunches" (BRIEF.md, Vision), and the "nothing gets thrown away" moment in the Experience section depends on it. Important rather than Core because the plan and grocery list function completely without it, but it is included at MVP because it is a defining part of the brief's stated experience, not a later refinement. [RESEARCH-INFORMED: added the finding that users of AI-personalized planners report plans ignoring leftovers carried into later days even on paid tiers (Samsung Food independent review, MEDIUM confidence)]

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

**Access:** Maya (Full on the Weekly Plan) and Sam can confirm or skip a leftover lunch per the Access Matrix in user-persona.md — for Sam, whose Weekly Plan access is View, marking a leftover lunch as eaten or skipped is a status update on the plan, not a change to it; the Later-phase older-kid login has View access. Jordan as a young kid profile has no login. Riley (Operator, support) has View access from v1. An unauthorized visitor cannot see it. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** N/A — the suggestion appears within the plan itself; no separate notification is sent.

**Data Notes:** Displayed: the leftover lunch suggestion attached to a day. Derived: which dinners produce leftover-worthy portions and which following day to suggest, computed from Planned Meal data. Source: AI Weekly Dinner Plan Generation (FEAT-03) output.

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) for the dinners it extends; affected by One-Tap Meal Swap (FEAT-04) when a source dinner is swapped.

**Signals:** leftover_lunch_suggested, leftover_lunch_confirmed, leftover_lunch_skipped.

### Meal Rating & Preference Learning

**ID:** FEAT-12

**Description:** Household members rate meals with a simple thumbs up or down after dinner, and future plans learn from those ratings to better match what the family actually likes.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief states the plan "learns what the family actually likes" over time and depicts kids giving "a thumbs up or down" after dinner (BRIEF.md, Vision, The Experience). Phased to v1 because meaningful learning requires a history of ratings that does not exist in a household's first weeks; MVP plan generation must work well from ratings alone being absent. [MODIFIED: phase changed to MVP (see Phase) — rating capture must start at launch for that history to exist; rating is available on both tiers, while learning from ratings is a paid-tier benefit (BRIEF.md, Business Context)]

**Connected Entities:** Rating (create), Dietary Rule (update — for learned dislikes)

**Key Capabilities:**
- Rate a meal — Household member gives a thumbs up or down after a dinner is cooked
- See ratings reflected in future plans — Meals rated poorly by the household appear less often; well-liked meals appear more often
- Rate individually — Each household member's rating is recorded separately, so the plan can learn different preferences per person
- Rate for a young kid — An adult hands their phone round the table and records each young kid profile's thumbs up or down on that kid's behalf [AUDIT-ADDED: 1 -- journey walk: BRIEF.md shows kids rating after dinner, but young kid profiles have no login]

**Primary Flows & Alternates:**
- Happy path: after dinner, household member gives a thumbs up or down -> the rating is recorded against that meal and that member -> future plan generation weights accordingly
- No rating given: a meal that goes unrated is simply treated as neutral — it is never assumed liked or disliked, and the household is never pressed to rate
- Repeated dislike: a meal rated down repeatedly by the same member is treated increasingly like a soft dislike, feeding back into that member's Dietary Rule data

**States:** Empty: a meal with no ratings yet shows the plain rating prompt with no history. Loading: submitting a rating confirms instantly. Error: a failed rating submission retries automatically in the background. Offline-degraded: a rating given offline is held locally and synced once connectivity returns.

**Validation & Limits:** One rating per household member per Planned Meal; a rating can be changed after submission up until the plan is archived. A rating recorded on behalf of a young kid profile counts as that kid's one rating for the meal.

**Access:** Maya (Ratings: Full) and Sam (Ratings: Own-only) each rate for themselves per the Access Matrix in user-persona.md, and either can record a rating on behalf of a young kid profile such as Jordan, who has no login. The Later-phase older-kid login rates on an Own-only basis. No one sees another member's individual rating broken out, only the aggregate effect on future plans. Riley (Operator, support) has View access from v1 for diagnosis. Rating works on both tiers; learning from ratings is paid. An unauthorized visitor cannot rate. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** N/A — rating is a self-initiated, in-app action with no notifications of its own.

**Data Notes:** Captured: a per-member, per-meal thumbs up/down. Displayed: N/A to other members individually (aggregate effect only, via future plans). Derived: the preference weighting applied during AI Weekly Dinner Plan Generation (FEAT-03), and any resulting soft-dislike update to Dietary Rule data. Source: household member input.

**Interactions:** Feeds AI Weekly Dinner Plan Generation (FEAT-03) directly and updates Dietary Rule data managed by Household Setup & Member Profiles (FEAT-01). Subscription & Billing Management (FEAT-14) gates the learning effect, not rating itself; ratings on manually planned weeks (FEAT-23) count too.

**Signals:** meal_rated_up, meal_rated_down, rating_changed, learned_dislike_applied. kid_rating_recorded_by_adult.

### Tonight's Dinner Reminder

**ID:** FEAT-13

**Description:** On the day of a planned dinner, the household gets a brief nudge with what's cooking and any prep reminder it needs (e.g., taking something out of the freezer).

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief depicts this directly: "On Wednesday at 5pm a nudge arrives: 'Tonight: 20-minute pasta — take the chicken out of the freezer.'" (BRIEF.md, The Experience). Included at MVP because it is core to closing the "what's for dinner?" problem the brief opens with, not a later refinement. [INFERRED: carried from Visionary draft]

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

**Access:** Each adult (Maya, Sam) controls their own nudge preference (Notification Prefs column of the Access Matrix in user-persona.md); neither kid row has notification settings; Riley (Operator, support) has none. An unauthorized visitor receives nothing. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** This feature is itself the communication: a device notification naming the day's dinner and any prep step. Where device notifications are not enabled, the day's dinner and prep step show as a "Tonight" card at the top of the plan instead of an email, keeping email to the weekly and account messages.

**Data Notes:** Displayed: N/A — this is a notification, not a data view. Derived: the prep-reminder text, computed from the Planned Meal's recipe requirements (e.g., a frozen ingredient). Source: AI Weekly Dinner Plan Generation (FEAT-03) and One-Tap Meal Swap (FEAT-04) output.

**Interactions:** Depends on AI Weekly Dinner Plan Generation (FEAT-03) and One-Tap Meal Swap (FEAT-04) for the meal it announces; its preference toggle lives within Household Setup & Member Profiles (FEAT-01). It also announces manually picked dinners from Manual Weekly Planning (FEAT-23).

**Signals:** dinner_nudge_sent, dinner_nudge_opened, dinner_nudge_correction_sent, dinner_nudge_disabled.

### Weekly Waste & Spend Check-In

**ID:** FEAT-25

**Description:** Once a week, the household can answer one quick, optional question about how much food it threw away and roughly what it spent on groceries, and see a simple trend of its own answers against where it started.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Success Criteria states "families say they throw away noticeably less food and spend less." The draft's food-waste metric depended on households reporting this, but no feature asked or held the answers. Important rather than Core because the plan-and-list loop works without it; MVP because a before-and-after comparison needs answers from the first weeks. Kept to one skippable question so it does not eat into the brief's under-10-minutes-a-week planning goal. [AUDIT-ADDED: 3 -- inverse entity check: the Reported Food Waste metric relied on self-reported data that no feature captured and no entity held; added with the Waste & Spend Check-In entity]

**Connected Entities:** Waste & Spend Check-In (create, read, update)

**Key Capabilities:**
- Answer the week's check-in — An adult picks "none", "a little" or "a lot" for food thrown away and can add a rough grocery spend
- Set a starting point — At the first check-in, the household says how much it typically threw away and spent before Plateful
- See the trend — The household sees its recent answers alongside its weekly budget and its starting point

**Primary Flows & Alternates:**
- Happy path: at the end of the week an in-app card asks the question -> an adult answers in two taps -> the trend updates
- Skipped week: a skipped week shows as a gap in the trend, with no follow-up reminder
- Correcting an answer: any adult can change the week's answer until the next check-in opens

**States:** Empty: before the first answer, the card explains what the check-in is for. Loading: N/A — answering is a single tap that confirms instantly. Error: a failed save is kept on the device and retried automatically. Offline-degraded: the check-in can be answered offline and syncs when connectivity returns.

**Validation & Limits:** One answer per household per week (the latest answer from any adult wins); spend is optional and must be a positive amount in the household's currency.

**Access:** Maya and Sam have Full access per the Waste Check-In column of the Access Matrix in user-persona.md; Jordan (young kid profile) and the Later-phase older-kid login have none; Riley (Operator, support) has none. An unauthorized visitor cannot see or answer it.

**Communications:** N/A — the check-in appears as an in-app card only, keeping notifications to the weekly plan and nightly nudge the brief describes.

**Data Notes:** Captured: the weekly waste answer, optional spend, and the one-time baseline. Displayed: the household's own trend. Derived: change against the baseline and against budget; aggregated across households only to measure product success, never sold or used for advertising (BRIEF.md, Business Context). Source: household input.

**Interactions:** Reads the weekly budget from Household Setup & Member Profiles (FEAT-01) and currency from Units, Currency & Locale Configuration (FEAT-16); reflects the effect of Pantry-Aware Suggestions (FEAT-05) and Leftover Rollover to Lunches (FEAT-11) without changing them.

**Signals:** checkin_shown, checkin_answered, checkin_skipped, checkin_baseline_set.

### Subscription & Billing Management

**ID:** FEAT-14

**Description:** The household can see its current plan tier, upgrade to the paid subscription to unlock the AI weekly plan and pantry-aware suggestions, and manage billing.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** The brief states the business model directly: a free tier for manual planning and the shared list, and "a paid household subscription (monthly or yearly)" that adds the AI plan, pantry suggestions, and learning (BRIEF.md, Business Context). Included at MVP because the product cannot generate any revenue, and the founder needs "paying households within about three months" (BRIEF.md, Constraints: Team/timeline), without it. [RESEARCH-INFORMED: added the finding that freemium with a low-cost annual household plan is the dominant model across all four profiled products, typically $10–$60 a year (vendor pricing pages, HIGH confidence)]

**Connected Entities:** Subscription (create, update)

**Key Capabilities:**
- View current tier — Household sees whether it is on the free or paid tier and what each includes
- Upgrade to paid — Organiser subscribes monthly or yearly to unlock AI features
- Manage billing — Organiser updates payment details and views billing history
- Downgrade or cancel — Organiser can move back to the free tier, with a clear explanation of what is lost
- Switch between monthly and yearly — Organiser changes billing period, taking effect at the next renewal [AUDIT-ADDED: 1 -- value-flow walk: the brief offers both periods but the draft had no way to move between them]

**Primary Flows & Alternates:**
- Happy path: organiser reviews what the paid tier unlocks -> chooses monthly or yearly -> subscribes -> AI plan generation and pantry-aware suggestions unlock immediately
- Payment failure: a failed renewal payment does not immediately cut off access — the household is given a clear grace-period notice and a chance to update payment details before losing paid features
- Downgrade: organiser downgrades to free; at the end of the paid period the household stops receiving new AI-generated plans, but every past plan, rating, recipe, pantry item, and the shared list stay fully available, and the current week carries on through Manual Weekly Planning (FEAT-23) [MODIFIED: the draft made the existing plan and list read-only after downgrade; changed based on documented backlash when a product retroactively paywalled users' own history (Cozi, Trustpilot average 2.1/5, HIGH confidence) — nothing a free household already had is taken away]
- Grace period ends unpaid: after the grace period the household reverts to the free tier exactly as in a downgrade; no data is removed [AUDIT-ADDED: 1 -- value-flow walk: the draft never said where a failed renewal ends]

**States:** Empty: N/A — every household has a tier from creation (defaulting to free). Loading: an upgrade action confirms within a few seconds. Error: a failed upgrade attempt preserves the chosen plan option and offers a retry rather than losing the selection. Offline-degraded: viewing current tier works offline; upgrading or changing billing requires connectivity.

**Validation & Limits:** Valid payment details required to upgrade; downgrade takes effect at the end of the current billing period, never mid-period without explanation. The payment grace period is 7 days; cancelling keeps paid features until the end of the period already paid for, and no partial refunds are made for the unused remainder of a period; all money flows are between the organiser and the product through the payment-processing capability — no money passes between households. [AUDIT-ADDED: 1 -- value-flow walk: the draft specified entry of money but not how it leaves (refund, forfeiture)]

**Access:** Only Maya (Organiser) has access to billing (Billing column: Full) per the Access Matrix in user-persona.md; Sam and both kid rows have none, though every adult member can see which tier the household is on. Riley (Operator, support) has View access to the plan tier only, from v1, and never to payment details. An unauthorized visitor cannot view or change billing. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** Sends upgrade/downgrade confirmations and a payment-failure grace-period notice to the organiser.

**Data Notes:** Captured: chosen tier and billing details. Displayed: current tier, what it includes, and billing history to the organiser. Derived: none. Source: organiser input at upgrade/downgrade time.

**Interactions:** Gates AI Weekly Dinner Plan Generation (FEAT-03), the plan-weighting part of Pantry-Aware Suggestions (FEAT-05), and the learning effect of Meal Rating & Preference Learning (FEAT-12); a downgraded household continues in Manual Weekly Planning (FEAT-23); Invite Another Household (FEAT-24) reads whether a referred household went on to pay.

**Signals:** subscription_upgraded, subscription_downgraded, subscription_payment_failed, subscription_cancelled. subscription_period_switched, subscription_grace_expired.

### Member Onboarding

**ID:** FEAT-15

**Description:** An adult who accepts a household invitation is guided from acceptance to seeing the current plan and grocery list for the first time.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** The brief's growth model depends on invited members having a smooth first experience — "growth is expected to come mostly from households inviting other households" (BRIEF.md, Business Context) is a household-to-household version of the same principle, and a confusing first join would undermine it. Included at MVP alongside Household Invitations (FEAT-09), since an invitation without a guided first-use is only half the feature. [INFERRED: carried from Visionary draft]

**Connected Entities:** Member Profile (read — the newly created one), Invitation (read — the accepted one)

**Key Capabilities:**
- Land in context — Newly joined member is taken directly to the current plan and grocery list, not a generic empty home screen
- Understand their role — Newly joined member sees a brief explanation of what they can do (view the plan, suggest swaps, shop, rate) versus what the organiser manages [MODIFIED: "swap" changed to "suggest swaps" to match the suggest-and-approve role split in One-Tap Meal Swap (FEAT-04)]

**Primary Flows & Alternates:**
- Happy path: invited adult accepts -> lands directly on the current week's plan with a short explanation of their role -> can immediately see and use the grocery list
- No active plan yet: if the household has no plan yet (e.g., organiser has not finished setup), the new member sees a short explanation of that state rather than a broken or empty view
- Re-join: a previously removed member who is re-invited goes through the same onboarding again rather than being silently restored to old data

**States:** Empty: covered above — a household with no plan yet shows an explained empty state, not a blank screen. Loading: N/A — landing in context is immediate on acceptance. Error: N/A — this is a one-time guided landing, not an operation that can fail independently of invitation acceptance itself. Offline-degraded: the landing view degrades to the standard offline plan/list view if there is no connectivity at the moment of acceptance.

**Validation & Limits:** Onboarding is shown exactly once per newly accepted invitation.

**Access:** Applies to any newly accepted Other Adult Member (Sam's role) per the Access Matrix in user-persona.md; the organiser is never shown this flow since they created the household themselves. Neither kid row goes through it — young kid profiles never accept invitations, and a Later-phase older-kid login would get its own short introduction with FEAT-17. Riley (Operator, support) has none. An unauthorized visitor never reaches it without a valid invitation. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** N/A — this is an in-app first-use experience, not a separate notification (the invitation itself, sent by FEAT-09, is the communication).

**Data Notes:** Displayed: the current Weekly Plan and Grocery List, plus a short role explanation. Derived: none. Source: existing household data.

**Interactions:** Depends on Household Invitations & Membership (FEAT-09) for the acceptance event it follows.

**Signals:** member_onboarding_started, member_onboarding_completed, member_onboarding_shown_empty_household.

### Invite Another Household

**ID:** FEAT-24

**Description:** Any adult in a household can share Plateful with another family through a personal invite link. When that family sets up its own household from the link, Plateful records which household invited them.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md, Business Context expects growth "mostly from households inviting other households," and BRIEF.md, Success Criteria states "most paying households were invited by another household." Household Invitations & Membership (FEAT-09) adds people to one household; nothing in the draft let one household bring in another or recorded where a new household came from, so the brief's growth criterion could be neither supported nor measured. Important rather than Core because the plan-and-list loop works without it; MVP because the first users are parents from the founder's kids' school and online parenting groups, where word of mouth starts on day one. No rewards or credits are attached — the brief names none. [AUDIT-ADDED: 3 -- inverse entity check: the household-to-household growth metric needs a referral record that no draft entity held; added with the Household Referral entity]

**Connected Entities:** Household Referral (create, read), Household (read — the inviting and the new household)

**Key Capabilities:**
- Share an invite link — An adult member shares a personal link through any messaging they already use
- Start a household from a link — A new family following the link begins its own household setup, with the referral recorded
- See who joined — The inviting member sees how many families have set up a household from their link

**Primary Flows & Alternates:**
- Happy path: Sam taps "invite another family" -> shares the link in the school parents' group chat -> a friend opens it and sets up their own household -> the referral is recorded against Sam's household
- Already has a household: someone who already belongs to a household opens the link and is told they already have one (one household per account in v1); nothing is recorded
- Delayed setup: the friend opens the link but finishes setup a few days later on the same device; the referral still counts if setup completes within 30 days

**States:** Empty: a household that has not invited anyone sees a short explanation and the share button. Loading: the link is ready instantly. Error: if the link cannot be created, a retry is offered. Offline-degraded: an already-created link can be copied offline; creating one requires connectivity.

**Validation & Limits:** One reusable personal link per adult member; a new household is attributed to at most one referring household; a household cannot refer itself.

**Access:** Maya and Sam have Full access per the Household Referrals column of the Access Matrix in user-persona.md; Jordan (young kid profile) and the Later-phase older-kid login have none; Riley (Operator, support) has none. An unauthorized visitor following a link sees only a welcome page describing Plateful with the inviter's first name and a start-setup option — never any other data about the inviting household.

**Communications:** The inviting member gets an in-app note when a family they invited finishes setting up; the link itself is sent by the member through their own messaging.

**Data Notes:** Captured: each member's link and each referral (inviting household, new household, date). Displayed: the number of families who joined from the member's link. Derived: whether a referred household went on to pay, and the share of new paying households that came through referrals. Source: link use at household creation and the new household's subscription tier.

**Interactions:** Depends on Household Setup & Member Profiles (FEAT-01) for the new household's creation and Subscription & Billing Management (FEAT-14) for whether it went on to pay; separate from Household Invitations & Membership (FEAT-09), which adds people to an existing household.

**Signals:** referral_link_shared, referral_link_opened, referred_household_created, referred_household_upgraded.

### Units, Currency & Locale Configuration

**ID:** FEAT-16

**Description:** The household can set the measurement units, currency, and supermarket aisle names that fit where they live, so the plan and grocery list read naturally for US and UK households alike.

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief states directly that "units (cups vs grams), currency and supermarket aisle names must be configurable, not hard-coded" for the US and UK launch markets (BRIEF.md, Scale & Non-Functional Expectations). Included at MVP because both markets are targeted from launch, not added later. [INFERRED: carried from Visionary draft]

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

**Access:** Only Maya (Organiser) can change these settings (Household Setup column: Full) per the Access Matrix in user-persona.md; Sam sees the results (View) but cannot change them; neither kid row has access; Riley (Operator, support) has View access from v1. An unauthorized visitor cannot see or change locale settings. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

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

**Rationale:** The brief names this directly — "older kids want to vote on dinners" (BRIEF.md, Target Users & Roles) — but also leaves kids' representation as an explicit open question, with the founder leaning toward "possibly a limited login for older kids later" (BRIEF.md, Open Questions). Phased to Later because it depends on resolving that open question about how older kids are represented at all. [INFERRED: carried from Visionary draft]

**Connected Entities:** Member Profile (read — the older-kid profile), Weekly Plan (read, update — the chosen option), Dinner Vote (create, read)

**Key Capabilities:**
- Vote on an option — Older kid picks a preferred dinner among a small set of safe alternatives for a given night
- See the outcome — Older kid sees which option was chosen once the organiser finalizes or the vote naturally resolves

**Primary Flows & Alternates:**
- Happy path: older kid is shown a small set of already-safety-checked options for a night -> votes for a preference -> the outcome is reflected in the plan
- No vote cast: a night with no vote cast simply proceeds with the AI's original suggestion — voting is an enhancement, never a blocker to having a plan
- Tie or no consensus: when multiple kids vote for different options, the organiser sees the split and makes the final call rather than the app arbitrarily deciding

**States:** Empty: a night with no voting round open shows nothing extra — this is normal. Loading: N/A — voting is a simple, instant tap. Error: a failed vote submission retries automatically. Offline-degraded: a vote cast offline is held locally and synced once connectivity returns.

**Validation & Limits:** Voting options are limited to a small set (2-3) that have already passed the allergy safety check; one vote per older-kid profile per voting round.

**Access:** Only the Later-phase older-kid limited login can vote (Dinner Voting column: Own-only) per the Access Matrix in user-persona.md; Maya (Full) opens voting rounds and makes the final call when votes conflict; Sam has View access to outcomes; Jordan as a young kid profile with no login has none; Riley (Operator, support) has none. An unauthorized visitor cannot vote. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

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

**Rationale:** The brief's privacy stance is explicit and repeated: "children's data is minimal, parent-controlled and never used for anything but the family's own plan" and "no ads, ever, and no selling of family data" (BRIEF.md, Constraints: Privacy). A stated privacy promise is not credible without a concrete way to exercise it, so this is included at MVP rather than deferred. [RESEARCH-INFORMED: added the finding that three meal-planning apps (Yummly, PlateJoy, Mealime) closed or were folded into other products within about two years (app-alternative reviews, MEDIUM confidence) — a readable export is also a credible answer to families' durability worries]

**Connected Entities:** Household (delete), Member Profile (update — own account; delete), Dietary Rule (delete), Support Request (create — general support contact)

**Key Capabilities:**
- Export household data — Organiser downloads a copy of the household's plans, ratings, and settings
- Delete a member profile — Organiser removes a member (including a kid profile) and their associated data
- Delete the household — Organiser permanently deletes the entire household and all its data
- Manage own account — Any adult member edits their own name and email, changes their sign-in, or deletes their own account [AUDIT-ADDED: 4 -- Settings and account management: the draft gave only the organiser account controls]
- Contact support — Any adult member sends a short description of a problem to the operator [AUDIT-ADDED: 4 -- Help and Guidance: Operator Read-Only Support Access (FEAT-22) assumes a household can report a problem, but no feature let them]

**Primary Flows & Alternates:**
- Happy path: organiser requests an export -> receives a complete, readable copy of the household's data within a reasonable time
- Member removal: organiser removes a kid or adult profile; that profile's dietary rules and ratings are deleted, and future plans no longer account for them
- Full deletion: organiser requests household deletion -> is shown clearly what will be lost and asked to confirm -> all household data, including every member profile, is permanently removed

**States:** Empty: N/A — this feature always has content to act on once a household exists. Loading: an export or deletion request shows clear progress rather than an indefinite wait. Error: a failed export or deletion is retried automatically or clearly reported, never left in an ambiguous half-deleted state. Offline-degraded: requesting export or deletion requires connectivity; the request queues if made offline and completes once reconnected.

**Validation & Limits:** Household deletion requires explicit confirmation of an irreversible action; export requests are rate-limited to a reasonable frequency to prevent abuse. Deleted data is removed within 30 days and is not kept in any form usable for other purposes; the organiser must hand over the role (FEAT-09) or delete the household before deleting their own account.

**Access:** Only Maya (Organiser) can export household data, remove members, or delete the household (Account & Data column: Full) per the Access Matrix in user-persona.md. Sam has Own-only access: he manages his own account details, can delete his own account, and can contact support. Neither kid row has access — kid profile data is managed by Maya. Riley (Operator, support) cannot see or trigger export or deletion. An unauthorized visitor has no access. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** Sends a confirmation once an export is ready or a deletion completes. A support contact sends the member an acknowledgement by transactional email.

**Data Notes:** Captured: none new. Displayed: an export file and deletion confirmations. Derived: the export file itself, compiled from all household records. Source: all existing household data.

**Interactions:** Reads and can remove data from Household Setup & Member Profiles (FEAT-01), and cascades to remove related Weekly Plan, Grocery List, and Rating data on full deletion.

**Signals:** data_export_requested, data_export_completed, member_deleted, household_deleted. own_account_updated, own_account_deleted, support_contacted.

## Nice-to-Have Features

### Weekly Plan History

**ID:** FEAT-19

**Description:** Household can look back at previous weeks' plans and grocery lists.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** User-Facing

**Rationale:** Not named directly in the brief, but a natural extension once several weeks of plans exist — useful for households wanting to repeat a past week or remember what they ate. Nice-to-Have because the core weekly loop (plan, swap, shop) functions completely without it. Phased to v1, once households have accumulated enough history for it to be useful. [INFERRED: carried from Visionary draft]

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

**Access:** Maya (Full, including re-using a past week), Sam (View), and the Later-phase older-kid login (View) per the Weekly Plan column of the Access Matrix in user-persona.md; Jordan as a young kid profile has no login; Riley (Operator, support) has View access from v1. An unauthorized visitor cannot see history. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

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

**Rationale:** The brief names this directly as "desirable later, not v1" (BRIEF.md, Ecosystem & Integrations: online grocery ordering with named retailers, which are left to Stage 4 per the functional-language rule) [MODIFIED: synthesis check 3 — named grocery retailers removed from the quoted brief text, since Stage 2 documents name capability categories only]. Phased to Later per the brief's own explicit timing; it is documented here rather than only as a deferral note because it is a genuine feature the product will eventually need, not merely an idea. [RESEARCH-INFORMED: added competitor context — Samsung Food integrates with 23 grocery retailers across 4 regions, while AnyList and Cozi offer none (Samsung Food profile, HIGH confidence); a differentiator, not a common feature, which supports the brief's Later timing]

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

**Access:** Maya and Sam have Full access to initiate a handoff per the Grocery List column of the Access Matrix in user-persona.md; the Later-phase older-kid login's Grocery List access covers adding and ticking only, not handoff; Jordan as a young kid profile has no login; Riley (Operator, support) has none. An unauthorized visitor cannot initiate a handoff. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

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

**Rationale:** The brief names this directly as "a nice-to-have for showing dinner on the family calendar, not v1" (BRIEF.md, Ecosystem & Integrations). Phased to Later exactly per the brief's own stated timing. [INFERRED: carried from Visionary draft]

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

**Access:** Only Maya (Organiser) can connect or disconnect the calendar capability, per the Household Setup column of the Access Matrix in user-persona.md; Sam and the kid rows see the resulting calendar entries only outside the product, on the calendar itself; Riley (Operator, support) has none. An unauthorized visitor has no access. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

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

**Rationale:** The brief names this directly: "the founder, as operator, needs only read-only support access to help a household. Nothing more." (BRIEF.md, Target Users & Roles). Nice-to-Have and phased to v1 because the product can launch and be supported manually at very small scale; a dedicated read-only view becomes worth building once household volume makes ad hoc support impractical. [INFERRED: carried from Visionary draft]

**Connected Entities:** Household (read), Member Profile (read — excluding kid profile dietary detail beyond what a specific report requires), Weekly Plan (read), Support Request (read, update — status and access record)

**Key Capabilities:**
- View household setup — Operator sees a household's setup and plan to diagnose a specific reported issue
- Nothing more — Operator cannot edit any household data, view billing detail beyond plan tier, or access kid profile data beyond what a specific safety report requires
- Open only against a request — Support access opens only for a household with an open Support Request, and closes when the request is resolved [AUDIT-ADDED: 4 -- Security and Privacy Posture: limits read-only access to the brief's "to help a household" purpose]
- Leave a visible record — Each time support views a household, the organiser can see when and why in the household's settings [AUDIT-ADDED: 4 -- Audit Logging: a "who looked, when" record supports the brief's privacy promise to parents]

**Primary Flows & Alternates:**
- Happy path: a household reports a problem -> operator opens read-only access to that specific household -> diagnoses the issue -> access is not used to make any change
- Scope creep prevention: the view itself has no edit controls at all, so there is no path by which read-only access could accidentally become a change
- Kid data restriction: a support case that does not involve a child's allergy safety never surfaces kid profile dietary detail, consistent with the brief's children's-privacy constraint

**States:** Empty: N/A — this feature has no content of its own beyond an existing household's data. Loading: N/A — a lightweight read-only view with no heavy computation. Error: N/A — a read failure simply shows nothing rather than partial or stale data. Offline-degraded: N/A — this is an operator-side tool requiring connectivity by nature.

**Validation & Limits:** Access is scoped to one household at a time, opened only in response to a specific support case; no bulk or cross-household browsing.

**Access:** Riley (Operator, support) uses this view (Support View column: Full — full use of a view that itself has no edit controls) with View access to Household Setup, Weekly Plan, Pantry, Grocery List, Recipe Library, Ratings, Safety Reports, and plan tier, per the Access Matrix in user-persona.md; no access to kid profile data except the allergy details inside a specific safety report, and none to Meal Swap, Notification Prefs, payment details, or edit actions of any kind. Maya has View access to the record of support visits. Sam and both kid rows have none. An unauthorized person cannot open support access. [MODIFIED: Access field realigned to the final Access Matrix, which splits kid profiles into a no-login young-kid row and a Later-phase older-kid row and adds the columns introduced during synthesis]

**Communications:** The organiser sees an in-app note each time support viewed the household, and is told when their Support Request is resolved. [AUDIT-ADDED: 4 -- Audit Logging]

**Data Notes:** Displayed: existing household data in read-only form. Derived: none. Source: existing household records.

**Interactions:** Reads from Household Setup & Member Profiles (FEAT-01), AI Weekly Dinner Plan Generation (FEAT-03), and Shared Grocery List (FEAT-06) without modifying any of them.

**Signals:** operator_support_view_opened, operator_support_view_closed. support_request_resolved.

## Feature Interaction Summary

| Feature | Depends On |
|---------|------------|
| FEAT-01 Household Setup & Member Profiles | None |
| FEAT-02 Dietary Rules & Allergy Safety Engine | FEAT-01 (dietary rule data) |
| FEAT-03 AI Weekly Dinner Plan Generation | FEAT-01, FEAT-02, FEAT-05, FEAT-08, FEAT-10, FEAT-12, FEAT-14, FEAT-16 |
| FEAT-04 One-Tap Meal Swap | FEAT-03 or FEAT-23 (the plan), FEAT-02 |
| FEAT-05 Pantry-Aware Suggestions | FEAT-14 (plan weighting is paid) |
| FEAT-06 Shared Grocery List | FEAT-03, FEAT-23, FEAT-04, FEAT-05, FEAT-16 |
| FEAT-07 Weekly Plan Ready Notification | FEAT-03 |
| FEAT-08 Recipe Library (Starter Recipes) | FEAT-02 (badges, ineligibility reasons) |
| FEAT-09 Household Invitations & Membership | FEAT-01 |
| FEAT-10 Recipe Import from Web Link | FEAT-08, FEAT-02 (badges) |
| FEAT-11 Leftover Rollover to Lunches | FEAT-03, FEAT-04 |
| FEAT-12 Meal Rating & Preference Learning | FEAT-03 or FEAT-23 (meals to rate), FEAT-01 (dietary rule updates), FEAT-14 (learning is paid) |
| FEAT-13 Tonight's Dinner Reminder | FEAT-03, FEAT-23, FEAT-04 |
| FEAT-14 Subscription & Billing Management | None |
| FEAT-15 Member Onboarding | FEAT-09 |
| FEAT-16 Units, Currency & Locale Configuration | None |
| FEAT-17 Older-Kid Dinner Voting | FEAT-03, FEAT-02 |
| FEAT-18 Account & Data Management | FEAT-01, FEAT-09 (organiser hand-over before own-account deletion) |
| FEAT-19 Weekly Plan History | FEAT-03, FEAT-06, FEAT-02 (re-check on reuse) |
| FEAT-20 Online Grocery Ordering Handoff | FEAT-06 |
| FEAT-21 Family Calendar Sync | FEAT-03, FEAT-04 |
| FEAT-22 Operator Read-Only Support Access | FEAT-01, FEAT-03, FEAT-06, FEAT-02 and FEAT-18 (support requests) |
| FEAT-23 Manual Weekly Planning | FEAT-08, FEAT-10, FEAT-02, FEAT-01 |
| FEAT-24 Invite Another Household | FEAT-01, FEAT-14 |
| FEAT-25 Weekly Waste & Spend Check-In | FEAT-01, FEAT-16 |


## How the Features Depend on Each Other

The dependency map's features table, navigation connections, cross-feature business rules and external touchpoints follow.

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


