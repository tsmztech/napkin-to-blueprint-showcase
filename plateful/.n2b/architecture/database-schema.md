---
document_type: database-schema
produced_by: schema-designer
status: final
stage: 4
database: Supabase (managed PostgreSQL), Pro plan, single region (US East) — technical-architecture.md Section 3, ADR "Database"
orm: Drizzle ORM with Drizzle Kit (generated SQL migrations), Supabase transaction-mode connection pooler — technical-architecture.md Section 3, ADR "ORM / Data Access"
entity_count: 17
table_count: 28
relationship_count: 42
created: 2026-09-28
---

# Database Schema Design -- Plateful

## 1. Schema Overview

- **Database:** Supabase-managed PostgreSQL (Pro plan), single region (US East). PostgreSQL is the reference dialect for every type in this document — no dialect translation is needed since the type-mapping table's Reference Mapping column applies directly.
- **ORM:** Drizzle ORM + Drizzle Kit, generating plain SQL migration files that also carry RLS policies, database functions (per-slot conditional writes, household sequence counters) and Realtime publication settings, reviewed and hand-edited where Drizzle Kit's diffing is weak (technical-architecture.md Section 3).
- **Tables:** 28 (18 domain-entity tables + 2 junction tables + 8 implicit/support tables, one of which — `allergens` — is a lookup table with 3 M:M/lookup-adjacent siblings folded into the "implicit" count for clarity; see the full breakdown in Section 2's table list and Section 9's counts).
- **Relationships:** 42 (38 one-to-many/one-to-one foreign-key relationships + 2 many-to-many junction tables + 2 self-referential/organiser-pointer relationships counted within the 38). See Section 3 for the itemized list.
- **Migration strategy:** Drizzle Kit-generated SQL migrations, applied via the CI pipeline described in technical-architecture.md Section 13 (dry-run on PR, applied to staging on merge to `main`, applied to production on a tagged release) — consistent with the "migration-based schema changes" ADR referenced there.

### Entity Map

**Domain Entities** (the 17 Stage 3 Shared Data Entities; `Dinner Vote` is realized as two physical tables, `dinner_vote_rounds` + `dinner_votes`, during normalization — see Section 6):

- `households` -> has many `member_profiles`, one `subscriptions`, many `weekly_plans`, many `grocery_lists`, many `pantry_items`, many `recipes` (imported), many `support_requests`, many `invitations`, many `household_referrals` (as referrer or referred), many `waste_spend_checkins`, many `household_aisles`
- `member_profiles` -> belongs to `households`; has many `dietary_rules`, `ratings`, `swap_suggestions`, `support_requests` (as raiser), `dinner_votes` (as voter), `push_subscriptions`; optionally one `households` row points back via `organiser_member_id`
- `dietary_rules` -> belongs to `member_profiles`; references `allergens`; has many `dietary_rule_change_history` rows
- `weekly_plans` -> belongs to `households`; has many `planned_meals`, one `grocery_lists`, many `dinner_vote_rounds`
- `planned_meals` -> belongs to `weekly_plans`; references `recipes`; has many `ratings`, `swap_suggestions`, `support_requests` (safety-concern subject); optionally links to another `planned_meals` row as its leftover-lunch source
- `recipes` -> optionally belongs to `households` (imported) or belongs to none (starter); has many `recipe_ingredients`; referenced by `planned_meals`, `dinner_vote_round_options`
- `pantry_items` -> belongs to `households`
- `grocery_lists` -> belongs to `households` and one `weekly_plans`; has many `grocery_list_items`
- `grocery_list_items` -> belongs to `grocery_lists`; references `household_aisles`; links to many `planned_meals` via `grocery_list_item_planned_meals`
- `ratings` -> belongs to `member_profiles` and `planned_meals`
- `invitations` -> belongs to `households`; references `member_profiles` (sent_by)
- `subscriptions` -> belongs to `households` (1:1); has many `payment_webhook_events`
- `swap_suggestions` -> belongs to `planned_meals` and `member_profiles` (suggesting_member)
- `dinner_vote_rounds` -> belongs to `weekly_plans`; has many `dinner_votes`; links to many `recipes` via `dinner_vote_round_options`
- `dinner_votes` -> belongs to `dinner_vote_rounds` and `member_profiles` (voter)
- `household_referrals` -> references `households` twice (referring_household_id, new_household_id)
- `support_requests` -> belongs to `households`; references `member_profiles` (raised_by) and optionally `planned_meals`; has many `support_access_sessions`
- `waste_spend_checkins` -> belongs to `households`

**Junction Tables:**
- `grocery_list_item_planned_meals` -> connects `grocery_list_items` <-> `planned_meals`
- `dinner_vote_round_options` -> connects `dinner_vote_rounds` <-> `recipes`

**Implicit Tables (discovered, not in Stage 3's Shared Data Entities list):**
- `recipe_ingredients` -> normalization split of `recipes.ingredients` (1NF)
- `household_aisles` -> normalization split + lookup table for `households.aisle_names` (ordered, >8 potential values)
- `allergens` -> lookup table for `dietary_rules.allergen` ("standard allergen list")
- `dietary_rule_change_history` -> normalization split of `dietary_rules.change_history`, survives parent deletion
- `support_access_sessions` -> normalization split of `support_requests.access_record`
- `push_subscriptions` -> named explicitly in technical-architecture.md Section 11 User Model Fields
- `payment_webhook_events` -> idempotent Stripe webhook ingestion (technical-architecture.md Payments & Billing ADR)
- `notification_dispatch_log` -> exactly-once/at-most-once dispatch tracking for FEAT-07 (plan-ready) and FEAT-13 (nudge + correction)

## 2. Table Definitions

### households

**Source:** Household entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-01.SPEC-003. Read by nearly every feature. Updated by FEAT-01, FEAT-16 (locale), FEAT-07 (plan-arrival), FEAT-09 (organiser field only), FEAT-14 (via subscriptions). Deleted by FEAT-18.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_name | VARCHAR(60) | NO | -- | CHECK (char_length(household_name) BETWEEN 1 AND 60) | Source field: "household_name" |
| organiser_member_id | UUID | YES | -- | FK -> member_profiles(id), UNIQUE, ON DELETE SET NULL, ON UPDATE CASCADE | Nullable only transiently during FEAT-01 setup (organiser profile created in the same transaction); XBR-15 guarantees exactly one organiser thereafter, enforced at the application layer via FEAT-09's hand-over transaction |
| weekly_budget | DECIMAL(10,2) | YES | -- | CHECK (weekly_budget >= 0) | Optional during partial setup; required before AI generation (FEAT-03.SPEC-009) enforced at the application layer |
| weekly_schedule_days | JSONB | YES | '{}'::jsonb | | Per-day time-constraint map (day -> minutes-or-null); attribute per the attribute-vs-entity rule (single value per household, never independently queried or referenced by other entities) |
| unit_system | VARCHAR(20) | NO | 'us_customary' | CHECK (unit_system IN ('us_customary','metric')) | Enum: 2 stable values, no metadata -- FEAT-16.SPEC-003 |
| currency | VARCHAR(3) | NO | 'USD' | CHECK (currency IN ('USD','GBP')) | Enum: supported set covering at least USD/GBP at launch (FEAT-16.SPEC-003); a lookup table would be preferred if the supported-currency set grows past 8 |
| plan_arrival_day | VARCHAR(10) | NO | 'sunday' | CHECK (plan_arrival_day IN ('monday','tuesday','wednesday','thursday','friday','saturday','sunday')) | FEAT-07.SPEC-004 |
| plan_arrival_time_slot | VARCHAR(20) | NO | 'evening' | CHECK (plan_arrival_time_slot IN ('morning','evening')) | Small set per FEAT-07.SPEC-004 |
| timezone | VARCHAR(64) | NO | 'America/New_York' | | Drives FEAT-03/FEAT-13 schedules (technical-architecture.md Section 11) |
| stripe_customer_id | VARCHAR(255) | YES | -- | UNIQUE | Organiser-controlled billing identity (technical-architecture.md Section 11); populated on first upgrade |
| status | VARCHAR(20) | NO | 'active' | CHECK (status IN ('active','closed')) | Active -> Closed set only by FEAT-18.SPEC-008 |
| deletion_requested_at | TIMESTAMPTZ | YES | -- | | Set when FEAT-18.SPEC-003 confirms deletion; the 30-day purge job hard-deletes the row (and its cascade) when this timestamp is 30 days old (FEAT-18.SPEC-008/010) |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | Updated on every write |

**Indexes:**
- `households_pkey` -- PRIMARY KEY (id)
- `households_organiser_member_id_key` -- UNIQUE (organiser_member_id)
- `households_stripe_customer_id_key` -- UNIQUE (stripe_customer_id)
- `households_deletion_requested_at_idx` -- INDEX (deletion_requested_at) WHERE deletion_requested_at IS NOT NULL -- driver: the 30-day purge job scans for households due for hard deletion (FEAT-18.SPEC-008)

---

### member_profiles

**Source:** Member Profile entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-01.SPEC-004/005 and FEAT-09.SPEC-007 (invitation acceptance). Read by FEAT-02, FEAT-03, FEAT-12, FEAT-15, FEAT-17, FEAT-22. Updated by FEAT-01, FEAT-09, FEAT-18, FEAT-07/FEAT-13 (own notification prefs). Deleted by FEAT-18 (hard) and FEAT-09 (soft, on self-leave).

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | Child cannot exist without parent household |
| auth_user_id | UUID | YES | -- | UNIQUE | References Supabase `auth.users(id)` (managed outside this schema); NULL for young-kid profiles with no login (technical-architecture.md Section 11) |
| display_name | VARCHAR(80) | NO | -- | CHECK (char_length(display_name) BETWEEN 1 AND 80) | First name or nickname only for kids (ASMP-26) |
| member_type | VARCHAR(10) | NO | -- | CHECK (member_type IN ('adult','kid')) | Enum: 2 stable values |
| household_role | VARCHAR(20) | NO | -- | CHECK (household_role IN ('organiser','adult','kid')) | Exactly one 'organiser' per household enforced at the application layer (XBR-15) plus a partial unique index below |
| age_band | VARCHAR(20) | YES | -- | | Kid profiles only; required for kids at the application layer (FEAT-01.SPEC-014) |
| kid_login_enabled | BOOLEAN | NO | false | | Later-phase (FEAT-17); default false (technical-architecture.md Section 11) |
| parental_consent_confirmed_at | TIMESTAMPTZ | YES | -- | | Kid profiles only; required before a kid profile is created (FEAT-01.SPEC-007) |
| parental_consent_confirmed_by | UUID | YES | -- | FK -> member_profiles(id), ON DELETE SET NULL, ON UPDATE CASCADE | The organiser who confirmed consent |
| notification_preferences | JSONB | NO | '{"plan_ready": true, "nightly_nudge": true}'::jsonb | | Per-member plan-ready/nudge on-off (XBR-13); attribute (single value, never independently queried) |
| referral_code | VARCHAR(20) | YES | -- | UNIQUE | Adult members only; one reusable personal referral link per FEAT-24.SPEC-003; attribute, not a separate entity (single value per member, no independent lifecycle) |
| status | VARCHAR(20) | NO | 'active' | CHECK (status IN ('invited','active','left','removed')) | Invited -> Active (FEAT-01/FEAT-09) -> Left (FEAT-09 self-leave, soft, retained) or Removed (FEAT-18 organiser removal) |
| version | INTEGER | NO | 1 | | Optimistic locking -- technical-architecture.md Section 11 User Model Fields; Contention line: removal-vs-own-edit race resolves reject-with-refresh |
| removed_at | TIMESTAMPTZ | YES | -- | | Set on FEAT-18 organiser removal or FEAT-09 self-leave; the member_profiles row itself is hard-deleted within 30 days only for the FEAT-18 removal path (see Section 4) |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `member_profiles_pkey` -- PRIMARY KEY (id)
- `member_profiles_household_id_idx` -- INDEX (household_id) -- FK index; also the Member List screen's primary read pattern (FEAT-01.SPEC-004)
- `member_profiles_auth_user_id_key` -- UNIQUE (auth_user_id)
- `member_profiles_referral_code_key` -- UNIQUE (referral_code)
- `member_profiles_one_organiser_per_household_idx` -- UNIQUE (household_id) WHERE household_role = 'organiser' -- enforces XBR-15's "exactly one organiser" invariant at the database level
- `member_profiles_parental_consent_confirmed_by_idx` -- INDEX (parental_consent_confirmed_by) -- FK index

---

### dietary_rules

**Source:** Dietary Rule entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-01.SPEC-006. Read by FEAT-02, FEAT-03, FEAT-23, FEAT-04. Updated by FEAT-01 and FEAT-12 (learned soft dislikes). Deleted by FEAT-01 (with confirmation) and FEAT-18 (cascade).

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| member_profile_id | UUID | NO | -- | FK -> member_profiles(id), ON DELETE CASCADE, ON UPDATE CASCADE | Child cannot exist without its member |
| rule_kind | VARCHAR(20) | NO | -- | CHECK (rule_kind IN ('allergy','religious','vegetarian','dislike')) | Enum: 4 stable values, no metadata needed |
| strength | VARCHAR(4) | NO | -- | CHECK (strength IN ('hard','soft')) | Allergy/religious/vegetarian = hard; dislike = soft (FEAT-02.SPEC-006) |
| allergen_id | UUID | YES | -- | FK -> allergens(id), ON DELETE RESTRICT, ON UPDATE CASCADE | Required for rule_kind='allergy' at the application layer; RESTRICT so a referenced allergen can't be silently removed from the standard list |
| extra_ingredient | VARCHAR(120) | YES | -- | | Optional named extra ingredient alongside a standard allergen |
| origin | VARCHAR(10) | NO | 'entered' | CHECK (origin IN ('entered','learned')) | "Entered by the organiser or learned from ratings" |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `dietary_rules_pkey` -- PRIMARY KEY (id)
- `dietary_rules_member_profile_id_idx` -- INDEX (member_profile_id) -- FK index; also the safety engine's primary read pattern, run on every candidate recipe check (FEAT-02.SPEC-002, high fan-in "Read by" lifecycle)
- `dietary_rules_allergen_id_idx` -- INDEX (allergen_id) -- FK index

---

### dietary_rule_change_history

**Source:** Implicit (normalization split of Dietary Rule's `change_history` field, feature-dependency-map.md; retention note in FEAT-01 Entity-Lifecycle Coverage Matrix: "change_history entry is retained indefinitely" and "survives rule deletion")
**Lifecycle:** Created by FEAT-01.SPEC-006 on every dietary rule change. Read by FEAT-01.SPEC-010 (organiser trust/audit view). Never updated or deleted.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| dietary_rule_id | UUID | YES | -- | FK -> dietary_rules(id), ON DELETE SET NULL, ON UPDATE CASCADE | Nullable so the history row survives the rule's own deletion (FEAT-01 Entity-Lifecycle note) |
| member_profile_id | UUID | NO | -- | FK -> member_profiles(id), ON DELETE CASCADE, ON UPDATE CASCADE | Denormalized reference so history remains queryable even after dietary_rule_id is nulled; cascades with the member since the history is meaningless without a household context |
| changed_by | UUID | NO | -- | FK -> member_profiles(id), ON DELETE RESTRICT, ON UPDATE CASCADE | The organiser who made the change; RESTRICT to preserve the audit trail's actor even if that member is later removed (member removal in practice happens through the cascade above, not through this FK) |
| change_summary | TEXT | NO | -- | | What changed (rule_kind, strength, allergen, or removal), in plain terms |
| created_at | TIMESTAMPTZ | NO | now() | | The change timestamp itself |

**Indexes:**
- `dietary_rule_change_history_pkey` -- PRIMARY KEY (id)
- `dietary_rule_change_history_dietary_rule_id_idx` -- INDEX (dietary_rule_id) -- FK index
- `dietary_rule_change_history_member_profile_id_idx` -- INDEX (member_profile_id) -- driver: FEAT-01.SPEC-010's organiser trust/audit view lists a member's rule history

---

### allergens

**Source:** Implicit (lookup table for Dietary Rule's `allergen` field, "from a standard allergen list" -- FEAT-01 Shared Data Entities)
**Lifecycle:** Seeded once at launch content-maintenance time; read by every dietary-rule screen and the safety engine.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| name | VARCHAR(60) | NO | -- | UNIQUE | e.g. "Peanuts", "Tree nuts", "Milk" |
| sort_order | INTEGER | NO | 0 | | Display order in the allergen picker |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `allergens_pkey` -- PRIMARY KEY (id)
- `allergens_name_key` -- UNIQUE (name)

**Enum-vs-lookup rationale:** the standard allergen list (the 9-14 major allergens recognized in the US/UK) exceeds the guide's <8-value enum threshold and may need occasional additions without a code deploy, so a lookup table is used rather than a CHECK-constrained enum column.

---

### weekly_plans

**Source:** Weekly Plan entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-03.SPEC-003/004 and FEAT-23.SPEC-001. Read by FEAT-06, FEAT-07, FEAT-11, FEAT-13, FEAT-15, FEAT-17, FEAT-19, FEAT-21, FEAT-22. Updated by FEAT-03, FEAT-23, FEAT-04, FEAT-17. Deleted by FEAT-18 (cascade); otherwise archived.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| week_start_date | DATE | NO | -- | | The calendar week this plan covers |
| origin | VARCHAR(20) | NO | -- | CHECK (origin IN ('ai_generated','manually_built')) | Enum: 2 stable values |
| status | VARCHAR(20) | NO | 'generated' | CHECK (status IN ('generated','reviewed','approved','active','archived')) | |
| approved_at | TIMESTAMPTZ | YES | -- | | Organiser approval (once per week, FEAT-03.SPEC-008) or NULL if auto-adopted |
| approved_via | VARCHAR(20) | YES | -- | CHECK (approved_via IN ('organiser','auto_adopted')) | |
| estimated_total | DECIMAL(10,2) | YES | -- | CHECK (estimated_total >= 0) | Computed by FEAT-03.SPEC-006; NULL until generation/quantities are computed |
| over_budget_note | TEXT | YES | -- | | Shown when no safe week fits the budget |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `weekly_plans_pkey` -- PRIMARY KEY (id)
- `weekly_plans_household_id_idx` -- INDEX (household_id) -- FK index
- `weekly_plans_household_id_week_start_date_key` -- UNIQUE (household_id, week_start_date) -- one plan per household per week
- `weekly_plans_household_id_status_idx` -- composite INDEX (household_id, status) -- driver: the Weekly Plan View's "current active plan" read (FEAT-03.SPEC-001) and Weekly Plan History's chronological browse (FEAT-19.SPEC-001), both filtering by status

---

### recipes

**Source:** Recipe entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-08.SPEC-004 (starter) and FEAT-10.SPEC-002/003 (imported). Read by FEAT-02, FEAT-03, FEAT-04, FEAT-06, FEAT-23, FEAT-19. Updated by FEAT-08 (starter maintenance) and FEAT-10 (household edits). Deleted (hard) by FEAT-10; soft-archived by FEAT-08.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | YES | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | NULL for starter-library recipes (shared, no owning household); required for imported recipes |
| name | VARCHAR(255) | NO | -- | | |
| steps | TEXT | NO | -- | CHECK (char_length(steps) <= 20000) | Length-capped method text (FEAT-10.SPEC-007) |
| cook_time_minutes | INTEGER | NO | -- | CHECK (cook_time_minutes > 0) | Required for schedule fit |
| rough_cost | DECIMAL(10,2) | YES | -- | CHECK (rough_cost >= 0) | Shown in the household's currency |
| origin | VARCHAR(20) | NO | -- | CHECK (origin IN ('starter_library','imported')) | |
| source_url | TEXT | YES | -- | | Imported recipes only |
| prep_requirements | TEXT | YES | -- | | Early-prep needs (e.g. defrosting), used by FEAT-13's nudge |
| archived_at | TIMESTAMPTZ | YES | -- | | Starter-content soft archive only (FEAT-08.SPEC-004); imported recipes are hard-deleted instead (see Section 4) |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

*Note:* `dietary_badges` is never stored -- it is computed live per household by FEAT-02 at view time (feature-dependency-map.md, Recipe fields note; FEAT-08 Shared Entities note), so it is intentionally absent from this table.

**Indexes:**
- `recipes_pkey` -- PRIMARY KEY (id)
- `recipes_household_id_idx` -- INDEX (household_id) -- FK index
- `recipes_household_id_source_url_key` -- UNIQUE (household_id, source_url) WHERE source_url IS NOT NULL -- driver: FEAT-10.SPEC-006 Duplicate Import Detection, scoped per household
- `recipes_name_trgm_idx` -- GIN trigram INDEX (name) -- driver: FEAT-08.SPEC-001 Recipe Library Browse & Search, "search by recipe name or ingredient" (technical-profile.md Search signal: simple filter)
- `recipes_search_tsv_idx` -- GIN INDEX (to_tsvector('english', name || ' ' || steps)) -- driver: FEAT-08.SPEC-001 name/ingredient full-text search, per technical-architecture.md Section 4 Search ADR (`tsvector` over recipe name and ingredient names)

---

### recipe_ingredients

**Source:** Implicit (normalization split of Recipe's `ingredients` field -- 1NF: "each with quantity and unit", feature-dependency-map.md)
**Lifecycle:** Created and updated with their parent Recipe by FEAT-08.SPEC-004 / FEAT-10.SPEC-002/003/004. Read by FEAT-02 (safety check), FEAT-06 (list consolidation).

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| recipe_id | UUID | NO | -- | FK -> recipes(id), ON DELETE CASCADE, ON UPDATE CASCADE | Child cannot exist without its recipe |
| ingredient_name | VARCHAR(120) | NO | -- | CHECK (char_length(ingredient_name) >= 1) | |
| quantity | DECIMAL(10,3) | NO | -- | CHECK (quantity > 0) | |
| unit | VARCHAR(20) | NO | -- | | e.g. "cup", "g", "each" |
| sort_order | INTEGER | NO | 0 | | Preserves ingredient list order |
| is_complete | BOOLEAN | NO | true | | FEAT-02.SPEC-007's ingredient-data-completeness flag; false triggers the fail-closed exclusion |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `recipe_ingredients_pkey` -- PRIMARY KEY (id)
- `recipe_ingredients_recipe_id_idx` -- INDEX (recipe_id) -- FK index; also the safety engine's per-recipe ingredient read (FEAT-02.SPEC-002, run on every candidate check)

---

### planned_meals

**Source:** Planned Meal entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-03, FEAT-23, FEAT-11. Read by FEAT-06, FEAT-12, FEAT-13, FEAT-17, FEAT-21. Updated by FEAT-04, FEAT-23, FEAT-02, FEAT-11. Deleted by FEAT-23 (hard, clear a night) and FEAT-02 (soft, safety removal); cascade on household deletion.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| weekly_plan_id | UUID | NO | -- | FK -> weekly_plans(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| night | DATE | NO | -- | | At most one dinner per night, enforced by the unique index below |
| meal_kind | VARCHAR(20) | NO | 'dinner' | CHECK (meal_kind IN ('dinner','leftover_lunch')) | |
| recipe_id | UUID | NO | -- | FK -> recipes(id), ON DELETE RESTRICT, ON UPDATE CASCADE | RESTRICT: a recipe referenced by a planned meal (including past-week history) cannot be hard-deleted out from under it; FEAT-10's hard delete of an imported recipe is blocked at the application layer while history references exist, consistent with "past Planned Meals that used it are unaffected" (FEAT-10.SPEC-004 notes) |
| source_meal_id | UUID | YES | -- | FK -> planned_meals(id), ON DELETE SET NULL, ON UPDATE CASCADE | Leftover-lunch link to its one source dinner, no more than two days earlier (FEAT-11.SPEC-003) |
| vegetarian_option | BOOLEAN | NO | false | | Whether a shared meal carries a vegetarian variant |
| cook_time_minutes | INTEGER | YES | -- | CHECK (cook_time_minutes > 0) | Carried from the recipe, sized for the household at generation time |
| rough_cost | DECIMAL(10,2) | YES | -- | CHECK (rough_cost >= 0) | Carried from the recipe |
| safety_badge_checked_at | TIMESTAMPTZ | YES | -- | | Set when FEAT-02.SPEC-002 passes the candidate; NULL means not yet checked (never displayed unchecked, XBR-01) |
| status | VARCHAR(20) | NO | 'proposed' | CHECK (status IN ('proposed','picked','confirmed','swapped','removed_safety','cooked','suggested','eaten','skipped')) | Dinner statuses (proposed/picked/confirmed/swapped/removed_safety/cooked) and leftover-lunch statuses (suggested/eaten/skipped) share one column since meal_kind disambiguates which subset applies |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | Updated on every write (swap, status change) |

**Indexes:**
- `planned_meals_pkey` -- PRIMARY KEY (id)
- `planned_meals_weekly_plan_id_idx` -- INDEX (weekly_plan_id) -- FK index; also the Weekly Plan View's full-week read (FEAT-03.SPEC-001, FEAT-23.SPEC-001)
- `planned_meals_weekly_plan_id_night_key` -- UNIQUE (weekly_plan_id, night) WHERE meal_kind = 'dinner' -- at most one dinner per night (FEAT-23.SPEC-005)
- `planned_meals_recipe_id_idx` -- INDEX (recipe_id) -- FK index
- `planned_meals_source_meal_id_idx` -- INDEX (source_meal_id) -- FK index; driver: FEAT-11.SPEC-004's re-evaluation when a source dinner changes

---

### swap_suggestions

**Source:** Swap Suggestion entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-04.SPEC-002 and FEAT-23.SPEC-003. Read by FEAT-03 (plan review). Updated by FEAT-04.SPEC-003/005.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| planned_meal_id | UUID | NO | -- | FK -> planned_meals(id), ON DELETE CASCADE, ON UPDATE CASCADE | Targets one Planned Meal slot; child cannot exist without the slot |
| suggesting_member_id | UUID | NO | -- | FK -> member_profiles(id), ON DELETE CASCADE, ON UPDATE CASCADE | The other adult member (required) |
| proposed_recipe_id | UUID | NO | -- | FK -> recipes(id), ON DELETE RESTRICT, ON UPDATE CASCADE | A safety-checked recipe |
| outcome | VARCHAR(10) | NO | 'suggested' | CHECK (outcome IN ('suggested','accepted','declined','lapsed')) | |
| resolved_at | TIMESTAMPTZ | YES | -- | | Set when outcome leaves 'suggested' |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `swap_suggestions_pkey` -- PRIMARY KEY (id)
- `swap_suggestions_planned_meal_id_idx` -- INDEX (planned_meal_id) -- FK index; also FEAT-04.SPEC-003's pending-suggestions-on-the-plan read
- `swap_suggestions_suggesting_member_id_idx` -- INDEX (suggesting_member_id) -- FK index
- `swap_suggestions_one_open_per_member_per_slot_idx` -- UNIQUE (planned_meal_id, suggesting_member_id) WHERE outcome = 'suggested' -- enforces "at most one open suggestion per member per night" (FEAT-04.SPEC-010)

---

### dinner_vote_rounds

**Source:** Dinner Vote entity, feature-dependency-map.md Shared Data Entities (split during normalization -- see Section 6: the entity's "round (night + 2-3 options)" field is a repeating group)
**Lifecycle:** Created by FEAT-17.SPEC-004. Read by FEAT-03, FEAT-17.SPEC-001/003. Updated (resolution) by FEAT-17.SPEC-005.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| weekly_plan_id | UUID | NO | -- | FK -> weekly_plans(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| night | DATE | NO | -- | | |
| status | VARCHAR(10) | NO | 'open' | CHECK (status IN ('open','resolved')) | |
| resolution_kind | VARCHAR(20) | YES | -- | CHECK (resolution_kind IN ('unanimous','fallback_ai_suggestion','organiser_final_call')) | Set when status becomes 'resolved' |
| resolved_recipe_id | UUID | YES | -- | FK -> recipes(id), ON DELETE RESTRICT, ON UPDATE CASCADE | The winning option |
| resolved_by | UUID | YES | -- | FK -> member_profiles(id), ON DELETE SET NULL, ON UPDATE CASCADE | The organiser, only when resolution_kind = 'organiser_final_call' |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `dinner_vote_rounds_pkey` -- PRIMARY KEY (id)
- `dinner_vote_rounds_weekly_plan_id_idx` -- INDEX (weekly_plan_id) -- FK index
- `dinner_vote_rounds_weekly_plan_id_night_key` -- UNIQUE (weekly_plan_id, night) -- one round per night

---

### dinner_vote_round_options

**Source:** Implicit (junction table; normalization split of the round's "2-3 safety-checked options", M:M between a round and its candidate recipes)
**Lifecycle:** Created by FEAT-17.SPEC-004 when the round opens; never updated or deleted independently of the round.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| dinner_vote_round_id | UUID | NO | -- | FK -> dinner_vote_rounds(id), ON DELETE CASCADE, ON UPDATE CASCADE, PK (composite) | |
| recipe_id | UUID | NO | -- | FK -> recipes(id), ON DELETE RESTRICT, ON UPDATE CASCADE, PK (composite) | |

**Indexes:**
- `dinner_vote_round_options_pkey` -- PRIMARY KEY (dinner_vote_round_id, recipe_id)
- `dinner_vote_round_options_recipe_id_idx` -- INDEX (recipe_id) -- FK index

---

### dinner_votes

**Source:** Dinner Vote entity, feature-dependency-map.md Shared Data Entities (split during normalization -- individual vote rows)
**Lifecycle:** Created by FEAT-17.SPEC-001. Read by FEAT-17.SPEC-003. Never updated (a vote is cast once; only the round's resolution changes) or deleted.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| dinner_vote_round_id | UUID | NO | -- | FK -> dinner_vote_rounds(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| voter_member_id | UUID | NO | -- | FK -> member_profiles(id), ON DELETE CASCADE, ON UPDATE CASCADE | An older-kid limited-login profile |
| choice_recipe_id | UUID | NO | -- | FK -> recipes(id), ON DELETE RESTRICT, ON UPDATE CASCADE | Must be one of the round's options (enforced at the application layer against dinner_vote_round_options) |
| created_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `dinner_votes_pkey` -- PRIMARY KEY (id)
- `dinner_votes_round_id_voter_key` -- UNIQUE (dinner_vote_round_id, voter_member_id) -- one vote per older-kid profile per round (FEAT-17.SPEC-006)
- `dinner_votes_voter_member_id_idx` -- INDEX (voter_member_id) -- FK index

---

### pantry_items

**Source:** Pantry Item entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-05.SPEC-001 and FEAT-06 ("already have it"). Read by FEAT-03, FEAT-06. Updated by FEAT-05. Deleted by FEAT-05 (cleared); cascade on household deletion.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| item_name | VARCHAR(80) | NO | -- | CHECK (char_length(item_name) BETWEEN 1 AND 80) | Free text, no rigid schema (FEAT-05.SPEC-004) |
| added_by | UUID | YES | -- | FK -> member_profiles(id), ON DELETE SET NULL, ON UPDATE CASCADE | The member who logged it; survives that member being removed |
| status | VARCHAR(20) | NO | 'active' | CHECK (status IN ('active','used_removed')) | |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `pantry_items_pkey` -- PRIMARY KEY (id)
- `pantry_items_household_id_idx` -- INDEX (household_id) -- FK index; also the pantry list's primary read pattern (FEAT-05.SPEC-001)
- `pantry_items_household_id_item_name_active_key` -- UNIQUE (household_id, lower(item_name)) WHERE status = 'active' -- enforces FEAT-05.SPEC-003's duplicate-merge rule at the database level (a second insert conflicts and the application merges instead)

---

### grocery_lists

**Source:** Grocery List entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-06.SPEC-002. Read by FEAT-15, FEAT-19, FEAT-20, FEAT-22. Updated by FEAT-06 (recalculation). Archived by FEAT-06 at week end; deleted by FEAT-18 cascade.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| weekly_plan_id | UUID | NO | -- | FK -> weekly_plans(id), ON DELETE CASCADE, ON UPDATE CASCADE, UNIQUE | One grocery list per weekly plan |
| status | VARCHAR(20) | NO | 'generated' | CHECK (status IN ('generated','active','archived')) | |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `grocery_lists_pkey` -- PRIMARY KEY (id)
- `grocery_lists_household_id_idx` -- INDEX (household_id) -- FK index
- `grocery_lists_weekly_plan_id_key` -- UNIQUE (weekly_plan_id)

---

### household_aisles

**Source:** Implicit (normalization split of Household's `aisle_names` field + lookup table -- FEAT-16.SPEC-002 needs ordering and renaming, so it exceeds the enum-vs-lookup threshold for "additional attributes (sort order)")
**Lifecycle:** Seeded with defaults at household creation (FEAT-16.SPEC-003); renamed/reordered by FEAT-16.SPEC-002.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| name | VARCHAR(60) | NO | -- | CHECK (char_length(name) <= 60) | Aisle-name length limit (FEAT-16.SPEC-003) |
| sort_order | INTEGER | NO | 0 | | Display/grouping order |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `household_aisles_pkey` -- PRIMARY KEY (id)
- `household_aisles_household_id_idx` -- INDEX (household_id) -- FK index; also the grocery list's aisle-grouping read on every render (FEAT-06.SPEC-001)

---

### grocery_list_items

**Source:** Grocery List Item entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-06.SPEC-002 (plan-derived) and FEAT-06.SPEC-001/SPEC-007 (manual). Updated by FEAT-06.SPEC-001 (tick, edit, "already have it"). Deleted by FEAT-06.SPEC-001 (manual removal, hard); cleared/carried at list archive.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| grocery_list_id | UUID | NO | -- | FK -> grocery_lists(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| ingredient_name | VARCHAR(80) | NO | -- | CHECK (char_length(ingredient_name) BETWEEN 1 AND 80) | Manual items' required-field rule (FEAT-06.SPEC-007) |
| quantity | DECIMAL(10,3) | YES | -- | CHECK (quantity IS NULL OR quantity > 0) | Combined across dinners, sized for the household |
| unit | VARCHAR(20) | YES | -- | | |
| aisle_id | UUID | YES | -- | FK -> household_aisles(id), ON DELETE SET NULL, ON UPDATE CASCADE | From the household's aisle names; nullable for a not-yet-grouped manual item |
| origin | VARCHAR(10) | NO | -- | CHECK (origin IN ('plan_derived','manual')) | |
| added_by | UUID | YES | -- | FK -> member_profiles(id), ON DELETE SET NULL, ON UPDATE CASCADE | Manual items' "added by" member; NULL for plan-derived lines |
| ticked | BOOLEAN | NO | false | | Ticked or unticked; idempotent per FEAT-06.SPEC-008 |
| version | INTEGER | NO | 1 | | Optimistic-locking column considered and rejected -- see Section 4: FEAT-06.SPEC-008 states quantity-edit conflicts resolve last-write-wins, so this column is present only to support the idempotent-tick merge logic at the application layer, not for reject-with-refresh semantics |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `grocery_list_items_pkey` -- PRIMARY KEY (id)
- `grocery_list_items_grocery_list_id_idx` -- INDEX (grocery_list_id) -- FK index; also the list screen's full-list read on every render (FEAT-06.SPEC-001), a hub-adjacent screen per technical-profile.md's Hub Screens finding (Shared Grocery List is the product's one identified hub screen, reached from 3+ features)
- `grocery_list_items_aisle_id_idx` -- INDEX (aisle_id) -- FK index; also the aisle-grouped display's sort/group key
- `grocery_list_items_grocery_list_id_ingredient_name_manual_key` -- UNIQUE (grocery_list_id, lower(ingredient_name)) WHERE origin = 'manual' -- enforces FEAT-06.SPEC-007's manual duplicate-merge rule

---

### grocery_list_item_planned_meals

**Source:** Implicit (junction table; Grocery List Item "plan-derived items trace to one or more Planned Meals", feature-dependency-map.md Relationships -- a genuine M:M since a consolidated line can combine ingredients from several dinners)
**Lifecycle:** Created and rewritten by FEAT-06.SPEC-002 on every recalculation.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| grocery_list_item_id | UUID | NO | -- | FK -> grocery_list_items(id), ON DELETE CASCADE, ON UPDATE CASCADE, PK (composite) | |
| planned_meal_id | UUID | NO | -- | FK -> planned_meals(id), ON DELETE CASCADE, ON UPDATE CASCADE, PK (composite) | Child cannot exist without either side; a swap or safety removal that drops a dinner cascades to remove its trace, which is how FEAT-06.SPEC-002 knows to recompute the combined line |

**Indexes:**
- `grocery_list_item_planned_meals_pkey` -- PRIMARY KEY (grocery_list_item_id, planned_meal_id)
- `grocery_list_item_planned_meals_planned_meal_id_idx` -- INDEX (planned_meal_id) -- FK index

---

### ratings

**Source:** Rating entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-12.SPEC-001. Read by FEAT-03, FEAT-22. Updated by FEAT-12 (change) and FEAT-09 (anonymize on leave). Deleted by FEAT-18 (member removal, household deletion).

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| member_profile_id | UUID | YES | -- | FK -> member_profiles(id), ON DELETE SET NULL, ON UPDATE CASCADE | Nullable so a self-leaving member's rating can be anonymized in place (XBR-16) rather than deleted; FEAT-18's organiser-removal path deletes the row outright instead (see Section 4) |
| planned_meal_id | UUID | NO | -- | FK -> planned_meals(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| value | VARCHAR(4) | NO | -- | CHECK (value IN ('up','down')) | |
| recorded_by | UUID | YES | -- | FK -> member_profiles(id), ON DELETE SET NULL, ON UPDATE CASCADE | The adult who recorded a young kid's rating on their behalf, where applicable |
| is_anonymized | BOOLEAN | NO | false | | Set true when the rating member self-leaves (FEAT-09.SPEC-008); member_profile_id is cleared at the same time |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | Updated on every rating change |

**Indexes:**
- `ratings_pkey` -- PRIMARY KEY (id)
- `ratings_member_profile_id_planned_meal_id_key` -- UNIQUE (member_profile_id, planned_meal_id) WHERE member_profile_id IS NOT NULL -- one rating per member per meal (FEAT-12.SPEC-002)
- `ratings_planned_meal_id_idx` -- INDEX (planned_meal_id) -- FK index; also FEAT-03's paid-tier weighting read (aggregate, high fan-in)
- `ratings_member_profile_id_idx` -- INDEX (member_profile_id) -- FK index

---

### invitations

**Source:** Invitation entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-09.SPEC-001. Read by FEAT-15. Updated by FEAT-09 (accepted, revoked, expired, resent).

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| token | VARCHAR(64) | NO | -- | UNIQUE | The invitation's shareable-link token |
| contact_detail | VARCHAR(255) | NO | -- | | The invitee's contact detail (email) |
| status | VARCHAR(10) | NO | 'sent' | CHECK (status IN ('sent','accepted','revoked','expired')) | |
| sent_by | UUID | NO | -- | FK -> member_profiles(id), ON DELETE RESTRICT, ON UPDATE CASCADE | Always the organiser; RESTRICT because an invitation's sender is part of its permanent history |
| accepted_member_profile_id | UUID | YES | -- | FK -> member_profiles(id), ON DELETE SET NULL, ON UPDATE CASCADE | The Member Profile created on acceptance |
| expires_at | TIMESTAMPTZ | NO | -- | | Set to created_at + 14 days on creation (FEAT-09.SPEC-006) |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `invitations_pkey` -- PRIMARY KEY (id)
- `invitations_household_id_idx` -- INDEX (household_id) -- FK index; also the Invitations Manager's list-all read (FEAT-09.SPEC-001)
- `invitations_token_key` -- UNIQUE (token)
- `invitations_expires_at_idx` -- INDEX (expires_at) WHERE status = 'sent' -- driver: FEAT-09.SPEC-006's 14-day expiry scheduled job

---

### subscriptions

**Source:** Subscription entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-01.SPEC-011 (defaults to free). Read by FEAT-03, FEAT-05, FEAT-12, FEAT-24, FEAT-22. Updated by FEAT-14.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE, UNIQUE | 1:1 with household |
| tier | VARCHAR(10) | NO | 'free' | CHECK (tier IN ('free','paid')) | |
| billing_period | VARCHAR(10) | YES | -- | CHECK (billing_period IN ('monthly','yearly')) | NULL on the free tier |
| billing_state | VARCHAR(20) | NO | 'active' | CHECK (billing_state IN ('active','payment_failed','cancelled','reverted_to_free')) | |
| stripe_subscription_id | VARCHAR(255) | YES | -- | UNIQUE | |
| current_period_end | TIMESTAMPTZ | YES | -- | | Drives downgrade/cancel-at-period-end timing (FEAT-14.SPEC-005) |
| grace_period_ends_at | TIMESTAMPTZ | YES | -- | | Set on payment failure; 7-day window (FEAT-14.SPEC-007) |
| cancel_at_period_end | BOOLEAN | NO | false | | |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `subscriptions_pkey` -- PRIMARY KEY (id)
- `subscriptions_household_id_key` -- UNIQUE (household_id)
- `subscriptions_stripe_subscription_id_key` -- UNIQUE (stripe_subscription_id)
- `subscriptions_grace_period_ends_at_idx` -- INDEX (grace_period_ends_at) WHERE billing_state = 'payment_failed' -- driver: the grace-period-expiry scheduled job (FEAT-14.SPEC-007)

*Note:* Billing history (`billing_history` field in Stage 3) is intentionally **not** persisted as its own table -- FEAT-14.SPEC-003's "Billing history list" is explicitly "sourced from the Payment Processing Integration" (Stripe), not stored redundantly in this database. `payment_webhook_events` below is the local, idempotency-oriented log of inbound events, not a billing-history display source.

---

### payment_webhook_events

**Source:** Implicit (process-state storage for external-system reconciliation; technical-architecture.md Payments & Billing ADR: "signed webhooks ingested idempotently"; FEAT-14.SPEC-009 Inbound Events, FEAT-14.SPEC-007 payment-failure/renewal events)
**Lifecycle:** Created on every inbound Stripe webhook delivery; read once by the processing job; never updated after processing completes.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| subscription_id | UUID | YES | -- | FK -> subscriptions(id), ON DELETE SET NULL, ON UPDATE CASCADE | Nullable so an event for a not-yet-linked Stripe customer can still be recorded and reconciled |
| stripe_event_id | VARCHAR(255) | NO | -- | UNIQUE | Stripe's own event ID -- the idempotency key; webhooks are at-least-once and can arrive out of order (technical-architecture.md Section 4 Background Jobs feasibility note) |
| event_type | VARCHAR(60) | NO | -- | | e.g. "invoice.payment_failed", "customer.subscription.updated" |
| payload | JSONB | NO | -- | | Raw event payload for reconciliation/debugging |
| processed_at | TIMESTAMPTZ | YES | -- | | NULL until the idempotent handler completes |
| created_at | TIMESTAMPTZ | NO | now() | | Receipt timestamp |

**Indexes:**
- `payment_webhook_events_pkey` -- PRIMARY KEY (id)
- `payment_webhook_events_stripe_event_id_key` -- UNIQUE (stripe_event_id) -- the idempotency guard itself
- `payment_webhook_events_subscription_id_idx` -- INDEX (subscription_id) -- FK index
- `payment_webhook_events_unprocessed_idx` -- INDEX (created_at) WHERE processed_at IS NULL -- driver: the retry worker's backlog scan

---

### household_referrals

**Source:** Household Referral entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-24.SPEC-004. Read by FEAT-24 (count) and FEAT-14 (via subscriptions, whether referred household upgraded). Updated by FEAT-24.SPEC-005 (upgraded flag).

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| referring_household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | The household whose member shared the link |
| referring_member_id | UUID | NO | -- | FK -> member_profiles(id), ON DELETE CASCADE, ON UPDATE CASCADE | The member's reusable personal link (member_profiles.referral_code) that was followed |
| new_household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE, UNIQUE | A new household is attributed to at most one referring household (enforced by this UNIQUE) |
| upgraded | BOOLEAN | NO | false | | Derived from the new household's Subscription; updated by FEAT-24.SPEC-005 |
| created_at | TIMESTAMPTZ | NO | now() | | The date setup completed |

**Indexes:**
- `household_referrals_pkey` -- PRIMARY KEY (id)
- `household_referrals_referring_household_id_idx` -- INDEX (referring_household_id) -- FK index; also FEAT-24.SPEC-001's "count of families joined" read
- `household_referrals_new_household_id_key` -- UNIQUE (new_household_id) -- no self-referral / single-attribution guarantee (XBR-20), plus CHECK below
- `household_referrals_referring_member_id_idx` -- INDEX (referring_member_id) -- FK index

**Constraints:**
- `household_referrals_no_self_referral_check` -- CHECK (referring_household_id <> new_household_id)

---

### support_requests

**Source:** Support Request entity, feature-dependency-map.md Shared Data Entities
**Lifecycle:** Created by FEAT-02.SPEC-004 (safety concern) and FEAT-18.SPEC-005 (general support). Read by FEAT-01, FEAT-22. Updated by FEAT-22 (status, access_record).

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| kind | VARCHAR(20) | NO | -- | CHECK (kind IN ('safety_concern','general_support')) | |
| raised_by | UUID | NO | -- | FK -> member_profiles(id), ON DELETE RESTRICT, ON UPDATE CASCADE | The reporting adult; RESTRICT to preserve the request's permanent-history actor |
| planned_meal_id | UUID | YES | -- | FK -> planned_meals(id), ON DELETE SET NULL, ON UPDATE CASCADE | Required for safety concerns (enforced at the application layer); the record survives even if the referenced meal is later purged |
| recipe_id | UUID | YES | -- | FK -> recipes(id), ON DELETE SET NULL, ON UPDATE CASCADE | The safety-concern recipe |
| note | VARCHAR(500) | YES | -- | CHECK (note IS NULL OR char_length(note) <= 500) | |
| status | VARCHAR(20) | NO | 'raised' | CHECK (status IN ('raised','under_review','resolved')) | |
| resolved_at | TIMESTAMPTZ | YES | -- | | |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `support_requests_pkey` -- PRIMARY KEY (id)
- `support_requests_household_id_idx` -- INDEX (household_id) -- FK index; also FEAT-01.SPEC-010's "organiser sees open requests" read
- `support_requests_status_idx` -- INDEX (status) WHERE status IN ('raised','under_review') -- driver: FEAT-22.SPEC-001's Support Request Queue, listing open requests across households
- `support_requests_planned_meal_id_idx` -- INDEX (planned_meal_id) -- FK index
- `support_requests_raised_by_idx` -- INDEX (raised_by) -- FK index

---

### support_access_sessions

**Source:** Implicit (normalization split of Support Request's `access_record` field -- 1NF: "each time support viewed the household, when and why" is a repeating group; FEAT-22.SPEC-004 Support Access Session Logging)
**Lifecycle:** Created and closed by FEAT-22.SPEC-004. Read by FEAT-22.SPEC-003 and FEAT-01.SPEC-010 (organiser's view of the record). Never deleted.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| support_request_id | UUID | NO | -- | FK -> support_requests(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | Denormalized for the organiser's household-scoped read without a join through support_requests |
| operator_auth_user_id | UUID | NO | -- | | References Supabase `auth.users(id)` for the operator account (no FK -- operator identity is managed entirely in Supabase Auth, not in member_profiles, since the operator is never a household member) |
| started_at | TIMESTAMPTZ | NO | now() | | |
| ended_at | TIMESTAMPTZ | YES | -- | | NULL while the session is open |
| created_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `support_access_sessions_pkey` -- PRIMARY KEY (id)
- `support_access_sessions_support_request_id_idx` -- INDEX (support_request_id) -- FK index
- `support_access_sessions_household_id_idx` -- INDEX (household_id) -- FK index; also FEAT-01.SPEC-010's/FEAT-22.SPEC-003's household-scoped access-record read
- `support_access_sessions_open_idx` -- INDEX (household_id) WHERE ended_at IS NULL -- driver: FEAT-22.SPEC-006's "only one open household at a time" gating check

---

### push_subscriptions

**Source:** Implicit (named explicitly in technical-architecture.md Section 11 User Model Fields: "push_subscriptions(member_id, onesignal_subscription_id, platform) for adults only")
**Lifecycle:** Created/updated on device registration (FEAT-07.SPEC-005 Device-Notification Delivery Integration). Read by FEAT-07, FEAT-13, FEAT-04's notification sends.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| member_profile_id | UUID | NO | -- | FK -> member_profiles(id), ON DELETE CASCADE, ON UPDATE CASCADE | Adults only (kids and the operator are never push recipients, XBR-13) enforced at the application layer |
| onesignal_subscription_id | VARCHAR(255) | NO | -- | UNIQUE | |
| platform | VARCHAR(20) | NO | -- | CHECK (platform IN ('web_push','ios_pwa')) | Web Push, including the iOS home-screen-installed PWA route (technical-architecture.md Frontend Framework ADR) |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `push_subscriptions_pkey` -- PRIMARY KEY (id)
- `push_subscriptions_member_profile_id_idx` -- INDEX (member_profile_id) -- FK index
- `push_subscriptions_onesignal_subscription_id_key` -- UNIQUE (onesignal_subscription_id)

---

### notification_dispatch_log

**Source:** Implicit (process-state storage for exactly-once/at-most-once dispatch guarantees: FEAT-07.SPEC-003 "sent exactly once per household per week"; FEAT-13.SPEC-005 "once per household per day" nudge and "at most one" correction)
**Lifecycle:** Created at dispatch time by FEAT-07.SPEC-001 and FEAT-13.SPEC-001/003. Never updated or deleted; retained for audit and dedup.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| member_profile_id | UUID | YES | -- | FK -> member_profiles(id), ON DELETE CASCADE, ON UPDATE CASCADE | NULL for a household-level record used only for the dedup key (e.g. once-per-week plan-ready); set for a per-member delivery attempt |
| notification_kind | VARCHAR(30) | NO | -- | CHECK (notification_kind IN ('plan_ready','tonight_nudge','same_day_swap_correction')) | |
| dedup_key | VARCHAR(60) | NO | -- | | e.g. `{household_id}:{week_start_date}` for plan-ready, `{household_id}:{night}` for the nudge |
| channel | VARCHAR(10) | NO | -- | CHECK (channel IN ('push','email','in_app')) | |
| sent_at | TIMESTAMPTZ | NO | now() | | |

**Indexes:**
- `notification_dispatch_log_pkey` -- PRIMARY KEY (id)
- `notification_dispatch_log_dedup_key_kind_key` -- UNIQUE (notification_kind, dedup_key) -- the exactly-once/at-most-once guard itself (FEAT-07.SPEC-003, FEAT-13.SPEC-005)
- `notification_dispatch_log_household_id_idx` -- INDEX (household_id) -- FK index

---

### waste_spend_checkins

**Source:** Waste & Spend Check-In entity, product-features.md Domain Entity Inventory (the one Stage 3 entity with no Shared Data Entities subsection, per feature-dependency-map.md's own note -- fields below are drawn directly from FEAT-25's feature-overview.md Shared Entities section instead)
**Lifecycle:** Created and updated by FEAT-25.SPEC-001. Transitioned (Offered/Answered/Skipped, locked) by FEAT-25.SPEC-003. Read by FEAT-25.SPEC-002.

| Column | Type | Nullable | Default | Constraints | Notes |
|--------|------|----------|---------|-------------|-------|
| id | UUID | NO | gen_random_uuid() | PK | |
| household_id | UUID | NO | -- | FK -> households(id), ON DELETE CASCADE, ON UPDATE CASCADE | |
| week_start_date | DATE | NO | -- | | |
| waste_amount | VARCHAR(10) | YES | -- | CHECK (waste_amount IS NULL OR waste_amount IN ('none','a_little','a_lot')) | NULL while status = 'offered' |
| spend | DECIMAL(10,2) | YES | -- | CHECK (spend IS NULL OR spend >= 0) | Optional rough grocery spend |
| is_starting_point | BOOLEAN | NO | false | | True on the household's one-time first-check-in baseline record |
| starting_point_waste | VARCHAR(10) | YES | -- | CHECK (starting_point_waste IS NULL OR starting_point_waste IN ('none','a_little','a_lot')) | Set only when is_starting_point = true |
| starting_point_spend | DECIMAL(10,2) | YES | -- | CHECK (starting_point_spend IS NULL OR starting_point_spend >= 0) | Set only when is_starting_point = true |
| status | VARCHAR(10) | NO | 'offered' | CHECK (status IN ('offered','answered','skipped')) | |
| locked_at | TIMESTAMPTZ | YES | -- | | Set when the next week's check-in opens (FEAT-25.SPEC-003); an Answered record is editable only until this is set |
| created_at | TIMESTAMPTZ | NO | now() | | |
| updated_at | TIMESTAMPTZ | NO | now() | | Latest-write-wins on a same-week resubmission by a different adult (FEAT-25.SPEC-005) |

**Indexes:**
- `waste_spend_checkins_pkey` -- PRIMARY KEY (id)
- `waste_spend_checkins_household_id_week_start_date_key` -- UNIQUE (household_id, week_start_date) -- one record per household per week
- `waste_spend_checkins_household_id_idx` -- INDEX (household_id) -- FK index; also FEAT-25.SPEC-002's trend-view recent-weeks read

## 3. Relationships

| Parent | Child | Type | FK Column | FK Location | ON DELETE | ON UPDATE | Optional |
|--------|-------|------|-----------|-------------|-----------|-----------|----------|
| households | member_profiles | one-to-many | household_id | member_profiles | CASCADE | CASCADE | NO |
| member_profiles | households | one-to-one (pointer) | organiser_member_id | households | SET NULL | CASCADE | YES |
| member_profiles | dietary_rules | one-to-many | member_profile_id | dietary_rules | CASCADE | CASCADE | NO |
| allergens | dietary_rules | one-to-many | allergen_id | dietary_rules | RESTRICT | CASCADE | YES |
| dietary_rules | dietary_rule_change_history | one-to-many | dietary_rule_id | dietary_rule_change_history | SET NULL | CASCADE | YES |
| member_profiles | dietary_rule_change_history | one-to-many | member_profile_id | dietary_rule_change_history | CASCADE | CASCADE | NO |
| member_profiles | dietary_rule_change_history | one-to-many (actor) | changed_by | dietary_rule_change_history | RESTRICT | CASCADE | NO |
| households | weekly_plans | one-to-many | household_id | weekly_plans | CASCADE | CASCADE | NO |
| households | recipes | one-to-many | household_id | recipes | CASCADE | CASCADE | YES |
| recipes | recipe_ingredients | one-to-many | recipe_id | recipe_ingredients | CASCADE | CASCADE | NO |
| weekly_plans | planned_meals | one-to-many | weekly_plan_id | planned_meals | CASCADE | CASCADE | NO |
| recipes | planned_meals | one-to-many | recipe_id | planned_meals | RESTRICT | CASCADE | NO |
| planned_meals | planned_meals | one-to-many (self, leftover link) | source_meal_id | planned_meals | SET NULL | CASCADE | YES |
| planned_meals | swap_suggestions | one-to-many | planned_meal_id | swap_suggestions | CASCADE | CASCADE | NO |
| member_profiles | swap_suggestions | one-to-many | suggesting_member_id | swap_suggestions | CASCADE | CASCADE | NO |
| recipes | swap_suggestions | one-to-many | proposed_recipe_id | swap_suggestions | RESTRICT | CASCADE | NO |
| weekly_plans | dinner_vote_rounds | one-to-many | weekly_plan_id | dinner_vote_rounds | CASCADE | CASCADE | NO |
| recipes | dinner_vote_rounds | one-to-many | resolved_recipe_id | dinner_vote_rounds | RESTRICT | CASCADE | YES |
| member_profiles | dinner_vote_rounds | one-to-many (resolver) | resolved_by | dinner_vote_rounds | SET NULL | CASCADE | YES |
| dinner_vote_rounds | dinner_vote_round_options | many-to-many | -- | dinner_vote_round_options | CASCADE | CASCADE | -- |
| recipes | dinner_vote_round_options | many-to-many | -- | dinner_vote_round_options | RESTRICT | CASCADE | -- |
| dinner_vote_rounds | dinner_votes | one-to-many | dinner_vote_round_id | dinner_votes | CASCADE | CASCADE | NO |
| member_profiles | dinner_votes | one-to-many | voter_member_id | dinner_votes | CASCADE | CASCADE | NO |
| recipes | dinner_votes | one-to-many | choice_recipe_id | dinner_votes | RESTRICT | CASCADE | NO |
| households | pantry_items | one-to-many | household_id | pantry_items | CASCADE | CASCADE | NO |
| member_profiles | pantry_items | one-to-many (added_by) | added_by | pantry_items | SET NULL | CASCADE | YES |
| households | grocery_lists | one-to-many | household_id | grocery_lists | CASCADE | CASCADE | NO |
| weekly_plans | grocery_lists | one-to-one | weekly_plan_id | grocery_lists | CASCADE | CASCADE | NO |
| households | household_aisles | one-to-many | household_id | household_aisles | CASCADE | CASCADE | NO |
| grocery_lists | grocery_list_items | one-to-many | grocery_list_id | grocery_list_items | CASCADE | CASCADE | NO |
| household_aisles | grocery_list_items | one-to-many | aisle_id | grocery_list_items | SET NULL | CASCADE | YES |
| member_profiles | grocery_list_items | one-to-many (added_by) | added_by | grocery_list_items | SET NULL | CASCADE | YES |
| grocery_list_items | grocery_list_item_planned_meals | many-to-many | -- | grocery_list_item_planned_meals | CASCADE | CASCADE | -- |
| planned_meals | grocery_list_item_planned_meals | many-to-many | -- | grocery_list_item_planned_meals | CASCADE | CASCADE | -- |
| member_profiles | ratings | one-to-many | member_profile_id | ratings | SET NULL | CASCADE | YES |
| planned_meals | ratings | one-to-many | planned_meal_id | ratings | CASCADE | CASCADE | NO |
| member_profiles | ratings | one-to-many (recorded_by) | recorded_by | ratings | SET NULL | CASCADE | YES |
| households | invitations | one-to-many | household_id | invitations | CASCADE | CASCADE | NO |
| member_profiles | invitations | one-to-many (sent_by) | sent_by | invitations | RESTRICT | CASCADE | NO |
| member_profiles | invitations | one-to-many (accepted) | accepted_member_profile_id | invitations | SET NULL | CASCADE | YES |
| households | subscriptions | one-to-one | household_id | subscriptions | CASCADE | CASCADE | NO |
| subscriptions | payment_webhook_events | one-to-many | subscription_id | payment_webhook_events | SET NULL | CASCADE | YES |
| households | household_referrals | one-to-many (referring) | referring_household_id | household_referrals | CASCADE | CASCADE | NO |
| households | household_referrals | one-to-one (referred) | new_household_id | household_referrals | CASCADE | CASCADE | NO |
| member_profiles | household_referrals | one-to-many | referring_member_id | household_referrals | CASCADE | CASCADE | NO |
| households | support_requests | one-to-many | household_id | support_requests | CASCADE | CASCADE | NO |
| member_profiles | support_requests | one-to-many (raised_by) | raised_by | support_requests | RESTRICT | CASCADE | NO |
| planned_meals | support_requests | one-to-many | planned_meal_id | support_requests | SET NULL | CASCADE | YES |
| recipes | support_requests | one-to-many | recipe_id | support_requests | SET NULL | CASCADE | YES |
| support_requests | support_access_sessions | one-to-many | support_request_id | support_access_sessions | CASCADE | CASCADE | NO |
| households | support_access_sessions | one-to-many | household_id | support_access_sessions | CASCADE | CASCADE | NO |
| member_profiles | push_subscriptions | one-to-many | member_profile_id | push_subscriptions | CASCADE | CASCADE | NO |
| households | notification_dispatch_log | one-to-many | household_id | notification_dispatch_log | CASCADE | CASCADE | NO |
| member_profiles | notification_dispatch_log | one-to-many | member_profile_id | notification_dispatch_log | CASCADE | CASCADE | YES |
| households | waste_spend_checkins | one-to-many | household_id | waste_spend_checkins | CASCADE | CASCADE | NO |

**Summary:** 38 one-to-many/one-to-one foreign-key relationships + 2 many-to-many junction tables (each counted once in the type breakdown, twice above as their two FK legs) + 2 self-referential/pointer relationships already included in the 38 = **42 relationships** (36 one-to-many, 4 one-to-one, 2 many-to-many).

## 4. Data Lifecycle & Integrity Decisions

| Entity | Soft Delete | Audit Columns | Optimistic Locking | Retention/Purge |
|--------|-------------|---------------|--------------------|-----------------|
| households | Yes — `deletion_requested_at` marks intent immediately on confirmation; the row (and cascade) is hard-deleted by a scheduled job 30 days later, no restore path (evidence: FEAT-18.SPEC-003/008/010, "permanently removes... within 30 days") | No — Household Contention line: "only Maya (Organiser) can modify household settings," so no multi-role audit trail is needed (evidence: feature-dependency-map.md Household Contention) | No — Contention: "Concurrent edits can occur only when Maya is signed in on two devices... resolution is last-write-wins per setting" (evidence: feature-dependency-map.md Household Contention) | Indefinite until deletion requested; 30-day purge window on deletion (evidence: ASMP-24 "kept for the life of the account"; FEAT-18.SPEC-010) |
| member_profiles | Yes — two distinct soft-delete paths: `status='left'` (FEAT-09 self-leave, retained indefinitely, no restore) or `status='removed'` + `removed_at` (FEAT-18 organiser removal, dietary_rules/ratings cascade-deleted immediately per XBR-16, the row itself purged within 30 days) (evidence: FEAT-09.SPEC-008 "soft removal... retained indefinitely"; FEAT-18.SPEC-002/007/010 "hard delete... removed within 30 days") | Yes — `updated_by` omitted in favor of `version`-tracked writes; multiple roles modify the same profile (organiser edits any profile, the member edits their own) (evidence: feature-dependency-map.md Member Profile Contention: "Maya... edits any profile... while Sam... edits his own account") — see Notes | Yes — `version` (evidence: technical-architecture.md Section 11 User Model Fields names `version` explicitly; Contention: "a removal by Maya racing an own-account edit by Sam resolves reject-with-refresh") | Kept for the life of the account except the FEAT-18 hard-delete path's 30-day purge (evidence: ASMP-24; XBR-16) |
| dietary_rules | No — hard delete, with the change-history table (below) retaining the audit trail independently (evidence: FEAT-01 Entity-Lifecycle Coverage Matrix, "no restore path — a removed rule is gone, but its change_history entry is retained indefinitely") | Yes — `dietary_rule_change_history.changed_by` (evidence: "visible to the organiser," FEAT-01 Shared Data Entities Data Notes; children's-allergy audit purpose, ASMP-26/27) | No — Contention: "An allergy can never be dropped without Maya's explicit confirmation, so no concurrent path can silently remove a hard rule" — the confirmation gate, not optimistic locking, is the safeguard (evidence: feature-dependency-map.md Dietary Rule Contention) | Indefinite (evidence: ASMP-24; FEAT-01 Non-Functional Notes "every household's setup history... is kept for the life of the account") |
| dietary_rule_change_history | No — append-only, never deleted; explicit non-goal (evidence: FEAT-02 Non-Goals "Automatic purge or deletion of safety-concern records" applies by the same account-history-depth logic to change history) | N/A — the table is itself the audit record | N/A — append-only, no concurrent-edit surface | Indefinite, survives parent rule deletion (evidence: FEAT-01 Entity-Lifecycle Coverage Matrix) |
| recipes (starter) | Yes — `archived_at` (evidence: FEAT-08.SPEC-004 "soft archive: a retired starter recipe is hidden... no restore path... retirement is a permanent content-maintenance decision") | No — single content-maintenance actor (the seeding pipeline), no household-role modification (evidence: FEAT-08.SPEC-004, starter recipes "read-only for households") | No — no concurrent-edit surface for starter content (read-only for households) | Indefinite; retired content stays in the row, archived (evidence: FEAT-08 Non-Goals) |
| recipes (imported) | No — hard delete (evidence: FEAT-10.SPEC-004 "Hard delete: removing an imported recipe deletes it from the household's pool outright... no restore path... no retention window") | No — Contention: "resolution is reject-with-refresh on the second save" is the concurrency control, not an audit trail; only Maya and Sam (Recipe Library: Full) edit, and neither needs attribution beyond ownership (evidence: feature-dependency-map.md Recipe Contention) | No — reject-with-refresh at the application layer using `updated_at` as the staleness check, not a dedicated `version` column, since the guide reserves `version` for entities with an explicit Contention resolution naming it; Recipe's Contention names "reject-with-refresh on the second save" without a stated mechanism, so `updated_at`-based optimistic concurrency suffices | No retention window; immediate permanent removal (evidence: FEAT-10.SPEC-004; FEAT-10 Non-Goals "Automatic purge or retention window for a removed imported recipe") |
| recipe_ingredients | No — cascades with its parent recipe | No — no independent modification surface | No — modified only as part of a recipe update | Tied to parent recipe's retention |
| weekly_plans | Yes (archival, not deletion) — `status='archived'` at week end, hard-deleted only via household deletion cascade, no independent purge (evidence: FEAT-03 Entity-Lifecycle Coverage Matrix "Soft archival... no restore path is needed since Archived plans remain fully browsable... never hard-deleted except by household deletion") | No — Contention: only Maya (Organiser, Full) and automated processes write to the plan; Sam's writes are indirect via accepted suggestions (evidence: feature-dependency-map.md Weekly Plan Contention) | No — Contention: "Resolution is reject-with-refresh per night slot," which is enforced by `swap_suggestions`' partial-unique index and the concurrency lock (FEAT-04.SPEC-009) at the `planned_meals` level, not a plan-level version column (evidence: feature-dependency-map.md Weekly Plan Contention) | Indefinite (evidence: FEAT-03 Non-Functional Notes "every past week is retained indefinitely per SC-18") |
| planned_meals | Yes for the safety-removal path (`status='removed_safety'`, no restore); hard delete for the manual-clear path (evidence: FEAT-02 Entity-Lifecycle Coverage Matrix "Soft removal: status is set to Removed (safety)... no restore path"; FEAT-23 Entity-Lifecycle Coverage Matrix "Hard delete: the clear action removes the record entirely... no retention/purge concern applies because a cleared night is a future/current unlived slot") | No — the acting role is always the organiser or an automated safety process; no multi-actor accountability requirement is named | No — Contention: "slot changes are reject-with-refresh with at most one active swap per slot," enforced by FEAT-04.SPEC-009's concurrency lock at the application layer against `updated_at`, not a dedicated `version` column (evidence: feature-dependency-map.md Planned Meal Contention) | Completed-week meals retained for the life of the account as part of Weekly Plan history; a cleared (unlived) slot carries no retention concern (evidence: FEAT-23 Entity-Lifecycle Coverage Matrix) |
| swap_suggestions | No — always resolves to a terminal outcome, retained (evidence: FEAT-04 Entity-Lifecycle Coverage Matrix "No delete: a suggestion always ends in a terminal outcome... and is retained as part of the plan's permanent history") | No — single-actor creation (the suggesting member), single-actor resolution (the organiser); the `outcome` field itself is the accountability record | No — Contention: "first-decision-wins with reject-with-refresh," enforced by the `outcome='suggested'` partial-unique index acting as the concurrency gate rather than a version column (evidence: feature-dependency-map.md Swap Suggestion Contention) | Indefinite (evidence: FEAT-04 Non-Goals "Automatic purge of swap suggestion history") |
| dinner_vote_rounds / dinner_vote_round_options / dinner_votes | No — retained for the life of the account (evidence: FEAT-17 Entity-Lifecycle Coverage Matrix "No delete or archive spec exists for Dinner Vote — an explicit non-goal") | No — Maya opens/resolves rounds, each vote is self-attributed by `voter_member_id`; no separate audit column needed | No — Contention: "a vote cast after the round has already resolved is rejected with refresh," enforced by `status='open'` check at write time rather than a version column (evidence: feature-dependency-map.md Dinner Vote Contention) | Indefinite (evidence: FEAT-17 Non-Goals "Automatic purge of vote history") |
| pantry_items | No — hard delete on clear/used-up, no restore (evidence: FEAT-05 Entity-Lifecycle Coverage Matrix "Hard delete: clearing an item... permanently removes the row... no restore path") | No — Contention: "Maya and Sam (Pantry Input: Full) add and clear items... Resolution is merge," enforced by the partial-unique index on active item names, not an audit column (evidence: feature-dependency-map.md Pantry Item Contention) | No — merge-based conflict resolution (the unique index), not reject-with-refresh, so no version column is needed (evidence: feature-dependency-map.md Pantry Item Contention "Resolution is merge") | No retention window for a cleared item (evidence: FEAT-05 Non-Goals "Automatic removal of stale or unused pantry items" is explicitly not the product's model, and a cleared item is deleted outright, not retained) |
| grocery_lists | Yes — `status='archived'` at week end, retained, no independent purge (evidence: FEAT-06 Entity-Lifecycle Coverage Matrix "Soft archive at week end... retained in full... no restore path is needed... no purge policy applies while the account is active") | No — no multi-actor accountability requirement beyond the items it contains | No — plan-derived recalculation and manual edits merge rather than conflict at the list level (Contention: "Resolution is merge") | Indefinite (evidence: FEAT-06 Non-Functional Notes "every past week's list is kept for the life of the account") |
| household_aisles | No — hard delete/rename in place; no independent lifecycle beyond the household's own deletion (evidence: FEAT-16 Entity-Lifecycle Coverage Matrix, no delete/archive path named for aisle_names beyond the whole-household cascade) | No — Maya alone writes aisle settings (Access: Sam/Riley view-only) | No — Contention/States: "a failed save keeps the prior value active" is the concurrency safeguard, not a version column (evidence: FEAT-16 Side-Effect Inventory) | Tied to household retention |
| grocery_list_items | No — manual-remove is an immediate hard delete, no restore (evidence: FEAT-06 Entity-Lifecycle Coverage Matrix "Hard delete on manual removal... immediate, no restore path... a deliberate simplicity choice"); plan-derived lines are rewritten in place by recalculation | No — Contention: "Maya and Sam... tick, add, edit, and remove items live" with no accountability requirement beyond the `added_by` column already present for manual items | Yes, narrowly — the `version` column exists only to support FEAT-06.SPEC-008's idempotent-tick merge logic; quantity-edit conflicts explicitly resolve last-write-wins rather than reject-with-refresh, so `version` is not used for rejection, only for detecting a redundant duplicate tick (evidence: feature-dependency-map.md Grocery List Item Contention "ticks are idempotent... conflicting quantity edits are last-write-wins") | Retained for the life of the archived list; no purge (evidence: FEAT-06 Entity-Lifecycle Coverage Matrix) |
| ratings | No — deleted only via FEAT-18 cascade (member removal, household deletion); anonymized in place, not deleted, on self-leave (evidence: FEAT-12 Entity-Lifecycle Coverage Matrix "Hard delete on member removal or household deletion... on a member's own voluntary departure, ratings are retained but anonymized") | No — `recorded_by` already captures the proxy-rating actor; no separate audit trail is named | No — Contention: "Two adults recording the same young kid's rating for the same meal resolve last-write-wins," enforced by the one-rating-per-member-per-meal unique index, not a version column (evidence: feature-dependency-map.md Rating Contention) | Kept for the life of the account except the FEAT-18 cascade-delete path (evidence: FEAT-12 Non-Functional Notes "ratings are kept for the life of the household account... except where XBR-16 requires deletion or anonymization") |
| invitations | No — retained with terminal status, never deleted (evidence: FEAT-09 Entity-Lifecycle Coverage Matrix "Invitation records are never hard-deleted; every invitation is retained with its terminal status... as household history") | No — only Maya sends/revokes; no multi-actor accountability beyond `sent_by` | No — Contention: "reject-with-refresh: whichever state change lands first wins," enforced by the `status` CHECK transition at the application layer against the current row, not a version column (evidence: feature-dependency-map.md Invitation Contention) | Indefinite (evidence: ASMP-24; FEAT-09 Non-Goals "Hard deletion or purge of Invitation records") |
| subscriptions | No — never deleted directly, only via household cascade (evidence: FEAT-14 Entity-Lifecycle Coverage Matrix "Subscription is never deleted directly by this feature; it is removed only through the Household deletion cascade") | No — only Maya (Billing: Full) changes it | No — Contention: "reject-with-refresh: an organiser change submitted against a stale billing state is refused," enforced against `updated_at`/`billing_state` at the application layer rather than a dedicated version column, since renewal-outcome events (from `payment_webhook_events`) are the more frequent concurrent writer and are idempotency-keyed instead (evidence: feature-dependency-map.md Subscription Contention) | `billing_history` is sourced live from Stripe, not retained in this schema (see Section 2 note); retained for the life of the account otherwise (evidence: FEAT-14 Non-Functional Notes, SC-18) |
| payment_webhook_events | No — append-only, never deleted | No — system-generated, no role attribution needed | No — idempotency is enforced by the `stripe_event_id` UNIQUE constraint, not optimistic locking | Indefinite; the event log itself is small and doubles as a reconciliation record (evidence: technical-architecture.md Payments & Billing ADR, "signed webhooks ingested idempotently") |
| household_referrals | No — "the record is written once by the system... and is never edited by any role" beyond the `upgraded` flag (evidence: feature-dependency-map.md Household Referral Contention) | No — system-written, no role attribution beyond the FK columns already present | No — single-field (`upgraded`) system update with no concurrent-write surface | Indefinite, no purge (evidence: FEAT-24 Non-Goals "Deletion or purge of Household Referral records"); cascade behavior on a referenced household's deletion is owned by FEAT-18, implemented here as ON DELETE CASCADE |
| support_requests | No — "Support Requests are never deleted by this feature except as part of the full household-deletion cascade" (evidence: FEAT-18 Entity-Lifecycle Coverage Matrix) | No — Contention: "Maya and Sam... raise requests but do not edit them afterwards; only Riley... changes status" — single-actor-per-transition, no multi-role audit column needed beyond the existing `raised_by` | No — Contention: "None" (feature-dependency-map.md Support Request Contention: "Riley... changes status, one household at a time") | Indefinite (evidence: FEAT-02 Non-Goals "Automatic purge or deletion of safety-concern records"; FEAT-22 Non-Goals "Automatic purge of Support Request history") |
| support_access_sessions | No — append-only audit log, never deleted (evidence: FEAT-22 Non-Goals "no feature in the dependency map deletes or archives a Support Request" — the same account-history-depth logic covers its access sessions) | N/A — the table is itself the audit record (`operator_auth_user_id`, `started_at`, `ended_at`) | N/A — append-only | Indefinite (evidence: ASMP-24 account-lifetime history depth) |
| push_subscriptions | No — hard-replaced on re-registration, hard-deleted on unsubscribe; no product-defined retention narrative exists for this implicit entity | No — device-registration data, no role-audit need | No — last-write-wins re-registration | Tied to member retention; removed with the member on cascade |
| notification_dispatch_log | No — append-only dedup/audit log | No — system-generated | No — the UNIQUE(notification_kind, dedup_key) constraint is itself the concurrency guard | Indefinite; low volume (at most one plan-ready and one nudge+correction per household per period, per FEAT-07/FEAT-13 Non-Functional Notes) |
| waste_spend_checkins | No — "every past check-in answer is kept for the life of the household account... no delete, archive, or purge mechanism exists" (evidence: FEAT-25 Entity-Lifecycle Coverage Matrix) | No — Access: only Maya/Sam answer; no multi-role accountability beyond the latest-wins field values themselves | No — Validation & Limits: "the latest answer from any adult wins" is explicit last-write-wins, not reject-with-refresh (evidence: FEAT-25.SPEC-005) | Indefinite (evidence: scope-boundaries.md SC-18, cited directly in FEAT-25 Non-Goals) |

**Multi-tenancy:** Household-scoped isolation, functionally equivalent to multi-tenancy though the product is not organizational B2B software. BRIEF.md confirms "there is one household per account in v1" (SC-03) — so the isolation boundary is the **household**, not a separate account/organization construct. Every household-owned table carries `household_id` directly (or transitively through a parent FK, e.g. `planned_meals` via `weekly_plan_id` -> `weekly_plans.household_id`), which serves the same role a `tenant_id` column would in a B2B product; no separate generic `tenant_id` column is added since `household_id` already uniquely and unambiguously scopes every table. This is the mechanism technical-architecture.md Section 3's Database ADR names directly: "Supabase adds Postgres row-level security, which enforces household isolation and kid-data restrictions in the database itself," and Section 11's Role & Permission Mapping keys every RLS policy on the JWT's `household_id` claim. Section 5 below carries this scoping into every access policy.

## 5. Security & Access Model

**Enforcement model:** Row-level security (RLS) is the primary authorization mechanism at the data layer. The selected database (Supabase Postgres, technical-architecture.md Section 3) supports native Postgres RLS, and Section 3's Database ADR names it directly as "the strongest control for the Compliance/privacy signal." Section 11's Role & Permission Mapping supplies the role model below — RLS policies read the JWT's `household_role` and `household_id` claims (technical-architecture.md Section 11 Session Model), never inventing a role not named there. Every policy additionally scopes by `household_id` per the Section 4 multi-tenancy decision.

### Access Policies

Role names below are the Section 11 Role & Permission Mapping's application representations: `organiser`, `adult` (Other Adult Member), `kid` (young, no login — structurally excluded, no policy exists for this role on any table), `kid` with `kid_login_enabled=true` (older-kid limited login, Later-phase), and `operator` (via a SECURITY DEFINER masked view, never a direct table grant).

| Table | Operation | Role(s) | Policy / Rule Sketch |
|-------|-----------|---------|----------------------|
| households | SELECT | organiser, adult | `household_id = current_household()` (both Full/View per Access Matrix; write fields differ, not row visibility) |
| households | UPDATE | organiser | `household_id = current_household() AND household_role = 'organiser'` — Sam's View access means no UPDATE policy for `adult` |
| member_profiles | SELECT | organiser, adult, kid (own row only) | organiser/adult: `household_id = current_household()`; kid: `household_id = current_household() AND id = current_member()` (View is Full for organiser, but kid profile data restrictions apply at the column-masking level, not row visibility — see Sensitive Fields) |
| member_profiles | UPDATE | organiser (any row), adult (own row only) | organiser: `household_id = current_household()`; adult: `household_id = current_household() AND id = current_member()` — ownership predicate for Sam's Own-only access |
| dietary_rules | SELECT | organiser, adult (via their own household's members) | `member_profile_id IN (SELECT id FROM member_profiles WHERE household_id = current_household())` |
| dietary_rules | INSERT/UPDATE/DELETE | organiser | Same predicate, restricted to `household_role = 'organiser'` — Full is organiser-only per the Access Matrix (Dietary Rules Editor) |
| weekly_plans, planned_meals | SELECT | organiser, adult, kid (older, view) | `household_id = current_household()` (transitively through weekly_plan_id for planned_meals) |
| weekly_plans | UPDATE (approve) | organiser | Ownership predicate: `household_id = current_household() AND household_role = 'organiser'` |
| planned_meals | UPDATE (swap) | organiser (direct), adult (via accepted suggestion only, enforced by FEAT-04's write path, not a broader RLS grant) | `household_id = current_household() AND household_role = 'organiser'` for direct swap writes |
| recipes | SELECT | organiser, adult, kid (older), operator (via masked view) | starter (`household_id IS NULL`): all authenticated household members; imported: `household_id = current_household()` |
| recipes | INSERT/UPDATE/DELETE (imported only) | organiser, adult | `household_id = current_household()` — Recipe Library is Full for both roles (FEAT-10) |
| pantry_items | SELECT/INSERT/UPDATE/DELETE | organiser, adult | `household_id = current_household()` — Pantry Input is Full for both; older-kid Later-phase is None |
| grocery_lists, grocery_list_items | SELECT | organiser, adult, kid (older), operator (via masked view) | `household_id = current_household()` |
| grocery_list_items | INSERT/UPDATE/DELETE | organiser, adult, kid (older, add/tick only) | organiser/adult: `household_id = current_household()`; older-kid: same predicate plus an application-layer check limiting the write to `ticked` and new-row INSERT, never edit/remove of another member's line (Grocery List: Full for add/tick only, per FEAT-06.SPEC-009) |
| ratings | SELECT | organiser (aggregate only, never another member's individual rating — enforced at the application/view layer, not RLS row visibility, since the row itself must exist for the owning member's own read) | `household_id` (via planned_meal_id -> weekly_plan_id) scoping plus an application-layer rule that no query ever returns another member's individual `value` broken out (FEAT-12 Data Notes: "individual ratings are never shown broken out") |
| ratings | INSERT/UPDATE | organiser (own + proxy for kids), adult (own only) | Own-only ownership predicate on `member_profile_id = current_member()`, with a proxy exception the application layer grants only to `organiser`/`adult` rows for a `kid` target in the same household |
| invitations | SELECT/INSERT/UPDATE | organiser | `household_id = current_household() AND household_role = 'organiser'`; the public acceptance read (by `token`) bypasses household-membership RLS entirely via a `SECURITY DEFINER` function scoped to the token, never a raw table grant |
| subscriptions | SELECT | organiser (Full), adult (tier only — column-masked), operator (tier only, via masked view) | `household_id = current_household()`; payment fields restricted to organiser at the column-grant level (see Sensitive Fields) |
| subscriptions | UPDATE | organiser | `household_id = current_household() AND household_role = 'organiser'` |
| household_referrals | SELECT | organiser, adult (own household's referring role) | `referring_household_id = current_household()` |
| support_requests | SELECT/INSERT | organiser (Full), adult (Own-only on the safety-concern kind) | organiser: `household_id = current_household()`; adult: same plus `raised_by = current_member()` for their own reports |
| support_requests, support_access_sessions | UPDATE (status)/SELECT (operator) | operator | No direct table grant — a `SECURITY DEFINER` view (`operator_support_view`) exposes exactly one household at a time, gated by an open `support_requests` row and masked per Sensitive Fields below, per FEAT-22.SPEC-006's "one household at a time" and FEAT-22.SPEC-007's masking rule |
| household_aisles | SELECT | organiser, adult, kid (older), operator (via masked view) | `household_id = current_household()` |
| household_aisles | INSERT/UPDATE/DELETE | organiser | `household_id = current_household() AND household_role = 'organiser'` |
| waste_spend_checkins | SELECT/INSERT/UPDATE | organiser, adult | `household_id = current_household()` |
| dietary_rule_change_history, payment_webhook_events, notification_dispatch_log, push_subscriptions | SELECT | organiser (change history only), service role (the rest) | `dietary_rule_change_history`: `household_id = current_household() AND household_role = 'organiser'` per FEAT-01's "visible to the organiser"; the other three tables are written and read only by trusted server-side code (Inngest functions, webhook handlers) under the Supabase service role, never exposed to any client-side role — no client-facing RLS policy exists for them beyond `ENABLE ROW LEVEL SECURITY` with no permissive policy (default-deny) |
| dinner_vote_rounds, dinner_vote_round_options, dinner_votes | SELECT | organiser, adult, kid (older) | `weekly_plan_id -> weekly_plans.household_id = current_household()` |
| dinner_vote_rounds | INSERT/UPDATE (open/resolve) | organiser | Ownership predicate: `household_role = 'organiser'` |
| dinner_votes | INSERT | kid (older, own vote only) | `voter_member_id = current_member() AND household_role = 'kid' AND kid_login_enabled = true` |

### Sensitive Fields

| Table | Column(s) | Sensitivity (source) | Protection Note |
|-------|-----------|----------------------|-----------------|
| dietary_rules | allergen_id, extra_ingredient, rule_kind, strength | Health-adjacent personal data including children's allergy information — "the most sensitive data in the product" (feature-dependency-map.md, Dietary Rule Data Sensitivity; ASMP-26/27) | Encryption at rest (Supabase-managed Postgres disk encryption); never included in application logs or error payloads; the `operator_support_view` masks every column of this table except allergen facts inside the one open safety-concern `support_requests` row being reviewed (FEAT-22.SPEC-007) |
| member_profiles | display_name, age_band, parental_consent_confirmed_at/by (kid rows) | Children's data under children's-privacy-class protection — minimal, parent-controlled (ASMP-26/27) | The `operator_support_view` suppresses kid-row detail beyond what a specific open safety-concern `support_requests` row requires (FEAT-22.SPEC-007); masked in any analytics export (technical-architecture.md Analytics & Product Telemetry ADR: "no names, ages or dietary values in properties") |
| member_profiles | referral_code | Personal identifier used to attribute referred households | Not exposed to any role other than the owning member and the organiser; the public referral-welcome flow resolves it server-side via a `SECURITY DEFINER` function, never a direct client read of another member's code |
| subscriptions | stripe_subscription_id, grace_period_ends_at, current_period_end | Financial personal data — "visible only to Maya, never to Sam, kids, or Riley" (feature-dependency-map.md, Subscription Data Sensitivity) | Column-level grant restricts SELECT on these columns to `organiser`; `adult` and `operator` roles see only `tier` via a masked view (`subscriptions_tier_only`), consistent with "Riley sees plan tier only" |
| households | stripe_customer_id | Financial identifier | Column-level grant restricts SELECT to `organiser`, same masking approach as subscriptions |
| support_requests | note, planned_meal_id, recipe_id (kind = 'safety_concern') | May contain children's allergy details (feature-dependency-map.md, Support Request Data Sensitivity) | Encryption at rest; the `operator_support_view` is the only cross-household-boundary read path and is masked per FEAT-22.SPEC-007; every read through it is logged in `support_access_sessions` |
| invitations | contact_detail | Personal data of a non-member (feature-dependency-map.md, Invitation Data Sensitivity) | Never exposed to any role beyond the sending organiser; masked in analytics; used only to deliver the invitation (ASMP-14) |
| payment_webhook_events | payload | May contain payment-processor metadata | Restricted to the service role only (no client-facing RLS policy); retained for reconciliation, never surfaced in any UI |

## 6. Normalization Log

### Changes Applied

| Entity | Field | Violation | Resolution |
|--------|-------|-----------|------------|
| Recipe | ingredients (each with quantity and unit) | 1NF: repeating group | Moved to new table `recipe_ingredients` |
| Household | aisle_names (household's aisle groupings and order) | 1NF: repeating group; also exceeds the enum-vs-lookup 8-value/no-metadata threshold (needs sort_order) | Moved to new table `household_aisles` |
| Household | weekly_schedule (per-night time constraints) | Considered for 1NF split | Kept as a `JSONB` attribute (`weekly_schedule_days`) rather than a child table — per the attribute-vs-entity rule, it has no independent identity, is a single value per household, and is never referenced by or queried independently from other entities |
| Dietary Rule | allergen (from a standard allergen list) | Considered for enum-vs-lookup | Moved to new lookup table `allergens` — the standard allergen list exceeds 8 values and needs a stable, addable-without-deploy value set |
| Dietary Rule | change_history (who changed the rule and when) | 1NF: repeating group; also needed to survive parent deletion (an explicit product decision, not a normal cascade) | Moved to new table `dietary_rule_change_history` with a nullable, `SET NULL` FK back to `dietary_rules` |
| Support Request | access_record (each time support viewed the household, when and why) | 1NF: repeating group | Moved to new table `support_access_sessions` |
| Dinner Vote | round (a night and its 2-3 safety-checked options), voter, choice, resolution -- conflated round-level and vote-level facts in one Stage 3 entity | 2NF: `round`'s option set does not depend on the `voter`/`choice` key at all, and a round has many votes | Split into `dinner_vote_rounds` (round-level: night, status, resolution), `dinner_vote_round_options` (M:M junction: round <-> its 2-3 candidate recipes), and `dinner_votes` (one row per older-kid's individual vote) |
| Grocery List Item | "plan-derived items trace to one or more Planned Meals" | Implicit M:M relationship with no explicit join structure in Stage 3 | Added junction table `grocery_list_item_planned_meals` |
| Subscription | billing_history (visible to the organiser) | 3NF-adjacent: billing_history is not this system's data of record — FEAT-14.SPEC-003 states it is "sourced from the Payment Processing Integration" | Not persisted as a table; documented as a live read from Stripe in the `subscriptions` table note. `payment_webhook_events` is a separate, narrower idempotency log, not a billing-history store |

### Denormalization Candidates (for the implementing team)

| Pattern | Tables Involved | Rationale for Deferral |
|---------|----------------|----------------------|
| Weekly Plan View (FEAT-03.SPEC-001) renders each dinner's recipe, safety badge, pantry callout, and cost in one screen — a join across `weekly_plans`, `planned_meals`, `recipes`, `recipe_ingredients`, `dietary_rules`, and `pantry_items` | weekly_plans, planned_meals, recipes, recipe_ingredients, dietary_rules, pantry_items | Normalize first; measure before denormalizing. If p95 latency on this read misses the ASMP-23 "well under a minute" / near-instant budgets, the implementing team can add a materialized per-household "plan card" read model |
| Grocery List screen (FEAT-06.SPEC-001) renders each item's aisle, "added by" name, and originating meals — a join across `grocery_list_items`, `household_aisles`, `member_profiles`, and `grocery_list_item_planned_meals` | grocery_list_items, household_aisles, member_profiles, grocery_list_item_planned_meals | Same rationale; this is the product's one identified hub screen (technical-profile.md Complexity Metrics), so it is the most likely candidate to eventually warrant a denormalized read model, but Postgres composite indexes should be tried first |
| Recipe Library search (FEAT-08.SPEC-001) needs each recipe's per-household eligibility (safety pass/fail) alongside name/ingredient search, joining `recipes`, `recipe_ingredients`, and `dietary_rules` | recipes, recipe_ingredients, dietary_rules | Feasibility note in technical-architecture.md's Search ADR already anticipates this: "precomputed per-recipe allergen/diet-attribute arrays so the household's eligibility is computed in the same query" — normalize first, and only precompute the attribute arrays if measured search latency misses the ~1-second target |

## 7. Indexing Strategy

Baseline PK/FK/unique indexes are defined per table in Section 2. The indexes below are the read-pattern-driven additions beyond that baseline; each already appears in its table's **Indexes** block and is repeated here with its citation for a single consolidated view.

### Read-Pattern Indexes

| Index | Table | Columns | Type | Driver (one-line citation) |
|-------|-------|---------|------|----------------------------|
| grocery_list_items_grocery_list_id_idx | grocery_list_items | grocery_list_id | btree | Shared Grocery List is the product's one identified hub screen (technical-profile.md Complexity Metrics: 3+ inbound navigation connections), rendered on nearly every session |
| recipes_name_trgm_idx | recipes | name | GIN trigram | FEAT-08.SPEC-001 "search by recipe name or ingredient" (technical-profile.md Search signal: simple filter); ASMP-23 targets ~1-second results |
| recipes_search_tsv_idx | recipes | to_tsvector(name, steps) | GIN full-text | Same driver; matches technical-architecture.md Section 4's Search ADR ("tsvector over recipe name and ingredient names with a GIN index") |
| weekly_plans_household_id_status_idx | weekly_plans | household_id, status | btree composite | FEAT-03.SPEC-001's "current active plan" read and FEAT-19.SPEC-001's chronological history browse both filter by status |
| dietary_rules_member_profile_id_idx | dietary_rules | member_profile_id | btree | High fan-in "Read by" lifecycle: FEAT-02.SPEC-002 runs this read on every candidate recipe check across every plan-generation, swap, manual-pick, vote, and history-reuse path (XBR-01) |
| recipe_ingredients_recipe_id_idx | recipe_ingredients | recipe_id | btree | Same high-fan-in driver as above — the safety engine reads every candidate recipe's ingredients on the same paths |
| ratings_planned_meal_id_idx | ratings | planned_meal_id | btree | FEAT-03.SPEC-003's paid-tier weighting reads accumulated ratings on every plan generation (aggregate query) |
| support_requests_status_idx | support_requests | status (partial: open statuses) | btree | FEAT-22.SPEC-001 Support Request Queue lists open requests across households |
| support_access_sessions_open_idx | support_access_sessions | household_id (partial: open sessions) | btree | FEAT-22.SPEC-006's "only one open household at a time" gating check, evaluated on every operator-view attempt |
| households_deletion_requested_at_idx | households | deletion_requested_at (partial) | btree | FEAT-18.SPEC-008's 30-day purge scheduled job scans for households due for hard deletion |
| invitations_expires_at_idx | invitations | expires_at (partial: sent) | btree | FEAT-09.SPEC-006's 14-day invitation-expiry scheduled job |
| subscriptions_grace_period_ends_at_idx | subscriptions | grace_period_ends_at (partial) | btree | FEAT-14.SPEC-007's 7-day grace-period-expiry scheduled job |
| notification_dispatch_log_household_id_idx | notification_dispatch_log | household_id | btree | Dedup lookups on every FEAT-07/FEAT-13 dispatch attempt |
| payment_webhook_events_unprocessed_idx | payment_webhook_events | created_at (partial: unprocessed) | btree | The idempotent webhook-processing retry worker's backlog scan (technical-architecture.md Payments & Billing ADR) |

### Growth-Tier Notes

- **Read replica for the Recipe Library / safety-check hot path:** if `dietary_rules`/`recipe_ingredients` join volume grows past what a single primary handles at "several thousand households" scale (ASMP-24), a Postgres read replica for these read-heavy, low-write tables is a straightforward next step — trigger condition: measured p95 on the safety-check join exceeds the ASMP-23 responsiveness budget under load.
- **Partitioning `notification_dispatch_log` and `payment_webhook_events` by month:** both are append-only, low-per-row-value logs that grow linearly with household count and time; trigger condition: either table exceeds a size where routine vacuum/index maintenance becomes noticeable, well beyond the volumes projected in FEAT-07/FEAT-13/FEAT-14's Non-Functional Notes for the stated household scale.
- **Materialized read model for the Weekly Plan View / Grocery List screens:** noted as a denormalization candidate in Section 6; only pursue if measured latency, not row count, misses the ASMP-22/23 targets.

None identified beyond the above for any other table — no other read pattern in the Stage 3 specs or technical-profile.md's Complexity Metrics names a hub screen, search signal, or high-fan-in lifecycle that isn't already covered by the baseline or the indexes above.

## 8. Reference & Seed Data

### Enum / Lookup Values

| Table or Column | Values | Type | Notes |
|-----------------|--------|------|-------|
| allergens | Milk, Eggs, Fish, Crustacean shellfish, Tree nuts, Peanuts, Wheat, Soybeans, Sesame, Gluten (non-celiac framing), Shellfish (mollusks, distinct from crustacean) | lookup table | The 9 FDA major allergens plus common UK/EU additions (gluten, mollusks) to serve both launch markets (BRIEF.md Geography: US and UK); exceeds the 8-value enum threshold and needs occasional additions without a deploy |
| households.unit_system | us_customary, metric | enum column | 2 stable values, no metadata (FEAT-16.SPEC-003) |
| households.currency | USD, GBP | enum column | Supported set covering at least USD and GBP at launch (FEAT-16.SPEC-003); the implementing team adds values here, not via a lookup table, unless the supported-currency set exceeds 8 |
| households.plan_arrival_day | monday..sunday | enum column | Default: sunday (FEAT-07.SPEC-004) |
| households.plan_arrival_time_slot | morning, evening | enum column | Default: evening (Sunday evening by default per FEAT-01/FEAT-07) |
| member_profiles.member_type | adult, kid | enum column | |
| member_profiles.household_role | organiser, adult, kid | enum column | Exactly one 'organiser' per household (XBR-15, enforced by partial unique index) |
| member_profiles.status | invited, active, left, removed | enum column | |
| dietary_rules.rule_kind | allergy, religious, vegetarian, dislike | enum column | |
| dietary_rules.strength | hard, soft | enum column | |
| weekly_plans.status | generated, reviewed, approved, active, archived | enum column | |
| planned_meals.status | proposed, picked, confirmed, swapped, removed_safety, cooked, suggested, eaten, skipped | enum column | Dinner and leftover-lunch statuses share one column, disambiguated by meal_kind |
| swap_suggestions.outcome | suggested, accepted, declined, lapsed | enum column | |
| dinner_vote_rounds.status | open, resolved | enum column | |
| dinner_vote_rounds.resolution_kind | unanimous, fallback_ai_suggestion, organiser_final_call | enum column | |
| pantry_items.status | active, used_removed | enum column | |
| grocery_lists.status | generated, active, archived | enum column | |
| grocery_list_items.origin | plan_derived, manual | enum column | |
| ratings.value | up, down | enum column | |
| invitations.status | sent, accepted, revoked, expired | enum column | |
| subscriptions.tier | free, paid | enum column | |
| subscriptions.billing_period | monthly, yearly | enum column | |
| subscriptions.billing_state | active, payment_failed, cancelled, reverted_to_free | enum column | |
| support_requests.kind | safety_concern, general_support | enum column | |
| support_requests.status | raised, under_review, resolved | enum column | |
| waste_spend_checkins.waste_amount | none, a_little, a_lot | enum column | |
| waste_spend_checkins.status | offered, answered, skipped | enum column | |
| push_subscriptions.platform | web_push, ios_pwa | enum column | Matches technical-architecture.md's PWA/Web Push routing (Frontend Framework ADR) |
| notification_dispatch_log.notification_kind | plan_ready, tonight_nudge, same_day_swap_correction | enum column | |
| recipes.origin | starter_library, imported | enum column | |

### Development Seed Data Plan

| Entity | Record Count | Description |
|--------|-------------|-------------|
| allergens | 11 | The full standard allergen list (seed data, not test data — required for every environment including production) |
| recipes (starter) | 100 | Meets the success-metrics.md Recipe Library Coverage at Launch target ("no repeated dinners, at least one recipe per major cuisine style for a household with typical dietary rules"); each with 5-10 recipe_ingredients rows |
| households | 20 | 3 free-tier, 15 paid-tier, 2 in a payment-failed grace period, spanning 1-6 members each to match ASMP-24's household-size range |
| member_profiles | 60 | Across the 20 seed households: 20 organisers, 20 other-adult members, 15 young-kid (no-login) profiles, 5 older-kid limited-login profiles (Later-phase feature flag on) -- "3 sample users with different roles" per household on average |
| dietary_rules | 40 | A realistic mix of allergies (peanut, dairy, gluten), one religious rule (halal), a few per-person vegetarian settings, and several soft dislikes, concentrated on 10-12 of the seed households so the safety engine has real exclusion cases to exercise |
| weekly_plans + planned_meals | 20 households x 3 weeks each | Includes at least one plan per household with a swap, a safety removal, and a leftover-lunk link, so downstream features (grocery list, rating, history) have realistic source data |
| grocery_lists + grocery_list_items | 1 per seeded weekly_plan | Each with a mix of plan-derived and manually-added lines, some ticked |
| pantry_items | 30 | 1-3 per household with pantry activity |
| ratings | 80 | Spread across cooked meals in the seed plans, including a few repeated down-ratings on one household to demonstrate FEAT-12.SPEC-005's learned-dislike automation |
| invitations | 8 | A mix of sent, accepted, revoked, and expired, to exercise FEAT-09's state machine |
| subscriptions | 20 | One per seed household, matching the tier/grace-state mix above |
| household_referrals | 5 | A few referral chains, including one referred household that has since upgraded |
| support_requests | 4 | 2 safety-concern, 2 general-support, in a mix of raised/under_review/resolved |
| dinner_vote_rounds + options + votes | 3 rounds across 2 households | Older-kid limited login enabled on those households, one unanimous, one split routed to the organiser's final call |
| waste_spend_checkins | 3 per household with 3+ seeded weeks | Including a starting-point record, to exercise FEAT-25's trend calculation |

## 9. Anti-Pattern Validation

| # | Check | Result | Notes |
|---|-------|--------|-------|
| 1 | Every table has a primary key | PASS | All 28 tables carry a UUID `id` PK, except the two junction tables (`grocery_list_item_planned_meals`, `dinner_vote_round_options`), which use a composite PK on their two FK columns per the guide's junction-table pattern |
| 2 | Every relationship has a FK constraint | PASS | All 42 relationships in Section 3 carry an explicit FK with a stated ON DELETE/ON UPDATE behavior |
| 3 | Column types match data semantics | PASS | Money uses DECIMAL, timestamps use TIMESTAMPTZ, identifiers use UUID, free text is length-capped VARCHAR or unbounded TEXT per the type-mapping table; no column uses a generic type for semantically distinct data |
| 4 | Monetary values use DECIMAL (not INTEGER/FLOAT) | PASS | `weekly_budget`, `estimated_total`, `rough_cost`, `spend`, `starting_point_spend` all use DECIMAL(10,2) with a `>= 0` CHECK |
| 5 | No comma-separated value columns | PASS | Every discovered repeating group (recipe ingredients, aisle names, dietary-rule change history, support access records, round options, grocery-item-to-meal traces) was split into its own table during normalization (Section 6); `weekly_schedule_days` and `notification_preferences` remain JSONB by deliberate attribute-vs-entity decision, not as a comma-separated shortcut |
| 6 | FK columns are indexed | PASS | Every FK column in Section 2 has a corresponding INDEX or is covered by a UNIQUE constraint that also serves as an index (e.g. `subscriptions_household_id_key`) |
| 7 | Timestamps include timezone | PASS | Every timestamp column across all 28 tables uses TIMESTAMPTZ; the one DATE-typed column family (`night`, `week_start_date`) is intentionally date-only per its functional meaning (a calendar day/week, not a moment in time) |
| 8 | No unconstrained polymorphic associations | PASS | No table uses a `{parent_type, parent_id}` pattern; every relationship (including the leftover-lunch self-reference on `planned_meals` and the two-legged `household_referrals`) is an exclusive, explicitly-typed FK |
| 9 | No god tables (>20 columns) | PASS | The widest table, `member_profiles`, has 17 columns; `households` has 15; every other table has 14 or fewer |

