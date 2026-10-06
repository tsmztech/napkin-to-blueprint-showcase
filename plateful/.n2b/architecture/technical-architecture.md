---
document_type: technical-architecture
produced_by: technical-architect
status: final
stage: 4
prerequisites:
  - .n2b/architecture/technical-profile.md
  - .n2b/architecture/technology-landscape.md
  - .n2b/architecture/technical-feasibility.md
created: 2026-09-28
---

# Technical Architecture -- Plateful

## 1. Project Technical Profile

# Project Technical Profile

## 1. Scale Metrics

Feature count from the `FEAT-*` folders in `.n2b/specifications/` (`ls -d .n2b/specifications/FEAT-*/ | wc -l`); total specs from the spec files on disk (`find .n2b/specifications/FEAT-*/ -name "FEAT-*.SPEC-*.md" | wc -l`); spec types from spec frontmatter across all five types.

| Metric | Value |
|--------|-------|
| Total features | 25 |
| Total specs | 196 |
| Screen specs | 60 |
| Automation specs | 46 |
| Logic/Rule specs | 55 |
| Integration specs | 15 |
| Notification specs | 20 |
| User-Facing features | 15 |
| Platform features | 7 |
| Lifecycle features | 3 |

The five spec-type counts (60 + 46 + 55 + 15 + 20 = 196) sum to the total spec count from item 3 with no discrepancy. The three feature-type counts (15 + 7 + 3 = 25) sum to the total feature count with no discrepancy.

## 2. Complexity Metrics

Structural complexity metrics extracted from product-features.md, feature-dependency-map.md, and feature-overview.md files. All values are counted, not estimated.

| Metric | Value |
|--------|-------|
| Entity count | 17 |
| Inter-entity relationships | 54 |
| Cross-feature business rules (XBR) | 20 |
| Cross-feature touchpoint rows | 197 |
| Cross-feature integration density | 7.88 (197 / 25) |
| Navigation connections | 39 |
| Hub screens (3+ inbound connections) | 1 (Shared Grocery List -- FEAT-06, "shared grocery list" destination context, reached from FEAT-03, FEAT-23, and FEAT-15) |
| Average specs per feature | 7.84 (196 / 25) |

Entity count is `grep -c "^### Entity: " .n2b/features/product-features.md`. Inter-entity relationships is the sum of named entity-to-entity references across the 16 non-"None" `**Relationships:**` lines in feature-dependency-map.md's Shared Data Entities section (the seventeenth entity, Waste & Spend Check-In, has no Shared Data Entities subsection and is excluded from that section by the document's own note): Household 8, Member Profile 7, Dietary Rule 2, Weekly Plan 5, Planned Meal 6, Recipe 2, Pantry Item 3, Grocery List 3, Grocery List Item 2, Rating 3, Invitation 2, Subscription 1, Swap Suggestion 3, Dinner Vote 2, Household Referral 2, Support Request 3 = 54. XBR count is `grep -c "^| XBR-" .n2b/specifications/feature-dependency-map.md`. Cross-feature touchpoint rows is the sum of all Cross-Feature Touchpoints table rows across all 25 feature-overview.md files. Navigation connections is the row count of the Navigation Connections table in feature-dependency-map.md. Hub screens were identified by grouping the Navigation Connections table's (To Feature, To Context) pairs and counting inbound rows per pair; only the "FEAT-06 | shared grocery list" pair reaches the 3-inbound threshold (rows from FEAT-03, FEAT-23, and FEAT-15); all other (To Feature, To Context) pairs have exactly 1 inbound row, including the several distinct FEAT-03 destination contexts ("new week's plan," "tonight's dinner in the plan," "next week's plan pantry callout," "current week's plan," "AI plan (first plan on its way)"), which are counted as separate destinations because each names a different screen context.

## 3. Capability Signals

Capability signals detected by scanning all specs -- all five types, including the Integration specs' Capability Category / Data Exchanged / Inbound Events / Degradation Behavior sections and the Notification specs' Channels / Trigger / Delivery Rules sections -- plus BRIEF.md and assumptions-constraints.md for the demand-side signals. Grep matches on generic keywords (e.g. "map," "address," "route," "file," "live") that resolved on inspection to unrelated document vocabulary (a "dependency map" cross-reference, an "email address" field, a navigation "route to," an "export file," a hyperlink rendered "live") are not counted as signal evidence; only matches whose surrounding text describes the signal's actual behavior are cited below.

| Signal | Present | Detail |
|--------|---------|--------|
| Real-time | Yes | FEAT-03.SPEC-011 (Real-Time Plan Sync Integration -- plan updates propagate live across household members' devices); FEAT-06.SPEC-005 (Live Grocery List Sync -- ticks, adds, and edits propagate live across devices); FEAT-01.SPEC-004 (member list updates live when an invitation is accepted, "live-updating, not a snapshot") |
| Offline | Yes | FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing -- every setup screen saves drafts locally and resumes online or offline); FEAT-06.SPEC-008 (Offline Conflict Resolution Rules -- grocery list changes queue and sync when connectivity returns); ASMP-25 (assumptions-constraints.md, ## Non-Functional Expectations -- "the shared grocery list keeps working, with changes queued and synced later, when used in a supermarket with poor or no signal; an already-generated plan stays viewable offline") |
| File upload | No | No specs reference file/media handling. The "upload"/"attachment"/"file" keyword matches found resolve to unrelated content: FEAT-02.SPEC-002's "badge attachment" (a UI label, not a file), FEAT-18.SPEC-012's explicit statement that the export is *not* carried as an email attachment, and FEAT-18's export-file *download* (a generated data export, covered under Import/export, not a file/media upload). No spec references an image, photo, or media upload. |
| Complex forms (10+ fields) | No | No specs exceed 10 fields. The product's guided setup is deliberately split into multiple small, single-purpose screens rather than one large form: FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) has 1 budget field plus a per-day time-constraint toggle/selector pair; FEAT-01.SPEC-006 (Dietary Rules Editor) and FEAT-14.SPEC-003 (Billing & Payment Management) each list roughly 5-6 distinct interactive elements including action buttons, not 10+ data-entry fields. |
| Background processing | Yes | FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation); FEAT-07.SPEC-001 (Plan-Ready Notification Trigger, scheduled against the household's plan-arrival day/time); FEAT-13.SPEC-001 (Tonight's Nudge Trigger, daily); FEAT-06.SPEC-004 (Week Rollover & Carryover); FEAT-25.SPEC-003 (Weekly Check-In Cycle); FEAT-09.SPEC-006 (Invitation Expiry, 14-day scheduled expiry); FEAT-04.SPEC-005 (Suggestion Lapse, scheduled against the night passing) |
| Authentication | Yes | Complexity: role-based. FEAT-01.SPEC-001 (Account Sign-Up & Sign-In); FEAT-01.SPEC-002 (Password Recovery); FEAT-01.SPEC-016 (Household Setup Authorization Rules, governs who can view or change each part of setup by role: Organiser, Other Adult Member, kid profile); FEAT-18.SPEC-011 (Account & Data Authorization Rules); role-based access matrices recur throughout the Shared Data Entities Contention notes (Maya/Organiser: Full, Sam/Other Adult Member: View or Own-only, Riley/Operator: read-only against an open Support Request, kid profiles: no login or limited login) |
| Search | Yes | Complexity: simple filter. FEAT-08.SPEC-001 (Recipe Library Browse & Search -- "Searching by recipe name or ingredient, and filtering by dietary badge"). No spec references full-text search or faceted search. |
| Payments/billing | Yes | FEAT-14.SPEC-001 through FEAT-14.SPEC-012 (Subscription & Billing Management -- tier overview, upgrade, billing/payment management, downgrade/cancel, billing state and refund rules, tier and billing access authorization, payment-failure/grace-period handling, apply-subscription-change, payment-processing integration, billing confirmation and grace-period notifications, transactional billing email); FEAT-01.SPEC-011 (Default Subscription Provisioning, free-tier default); ASMP-33 (assumptions-constraints.md, ## Dependencies -- "Payment-processing capability -- Required to run the paid household subscription"); BRIEF.md, ## Scale & Non-Functional Expectations ("the AI cost per household must stay small... The free tier gets no AI") |
| Notifications (email/push/SMS) | Yes | 20 Notification specs across FEAT-01, FEAT-02, FEAT-04, FEAT-07, FEAT-09, FEAT-13, FEAT-14, FEAT-18, FEAT-22, FEAT-24 (e.g. FEAT-07.SPEC-002 Plan-Ready Notification Message, Channels: push, email; FEAT-13.SPEC-002 Tonight's Dinner Nudge Message); ASMP-31 (assumptions-constraints.md, ## Dependencies -- "Device-notification delivery capability -- Required for the weekly 'plan ready' notification, the daily 'tonight's dinner' nudge, and swap-suggestion alerts"); ASMP-32 ("Transactional email capability -- Required for account sign-up and sign-in recovery, the plan-ready email fallback, billing and grace-period notices, data-export and deletion confirmations, safety-concern reports reaching the operator, and support acknowledgements") |
| Third-party integrations | Yes | 15 Integration specs (FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-03.SPEC-010, FEAT-03.SPEC-011, FEAT-04.SPEC-007, FEAT-06.SPEC-005, FEAT-07.SPEC-005, FEAT-07.SPEC-006, FEAT-08.SPEC-004, FEAT-10.SPEC-005, FEAT-14.SPEC-009, FEAT-14.SPEC-012, FEAT-18.SPEC-012, FEAT-20.SPEC-002, FEAT-21.SPEC-002); feature-dependency-map.md External Touchpoints table names 9 capability categories (AI text/plan generation, device-notification delivery, transactional email, payment processing, recipe/food-content data, real-time data synchronization, web-page recipe extraction, online grocery ordering [Later], family calendar [Later]); BRIEF.md, ## Ecosystem & Integrations |
| AI/ML behavior | Yes | FEAT-03.SPEC-010 (AI Plan Generation Capability Integration); FEAT-04.SPEC-007 (Swap Alternatives Generation); FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation); FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule -- learned preference influences generation on the paid tier); FEAT-12.SPEC-005 (Repeated-Dislike Learned Update); ASMP-30 (assumptions-constraints.md, ## Dependencies -- "AI text/plan-generation capability -- Required to generate the weekly dinner plan and swap alternatives"); BRIEF.md, ## Ecosystem & Integrations ("AI model: generates the weekly plans and suggestions") |
| Geo/maps | No | No specs reference location or mapping behavior. The only word-boundary matches for "map" resolve to "dependency map" (a cross-reference to feature-dependency-map.md, unrelated to geographic mapping); no spec contains geolocation, GPS, proximity, or distance content. |
| Import/export | Yes | FEAT-10.SPEC-001 (Import by Link -- pasting a web link to import a recipe); FEAT-10.SPEC-005 (Web Page Recipe Extraction integration); FEAT-18.SPEC-001 (Export Household Data); FEAT-18.SPEC-006 (Export Generation Processing); BRIEF.md, ## Ecosystem & Integrations ("Recipe sources: a starter recipe library, plus saving recipes from any website by pasting a link"). No spec references CSV, spreadsheet, or bulk multi-record import/export -- the import/export surface is single-recipe web import and a whole-household data export. |
| Collaboration/concurrency | Yes | Extensive: FEAT-01.SPEC-003/004/005/006/008 (last-write-wins or reject-with-refresh on concurrent household/member/dietary-rule edits from two devices); FEAT-03.SPEC-011 and FEAT-06.SPEC-005/007/008 (live shared grocery list and plan, merge/last-write-wins/offline reconciliation); FEAT-04.SPEC-004 (Apply Meal Swap, reject-with-refresh, at most one active swap per slot); FEAT-09.SPEC-003/004 (organiser hand-over, requires acceptance so only one organiser exists at a time); FEAT-10.SPEC-004 (Edit Imported Recipe, reject-with-refresh); FEAT-12.SPEC-002 (Rating Submission, last-write-wins for two adults recording the same kid's rating); FEAT-14.SPEC-003 (Billing & Payment Management, reject-with-refresh against stale billing state); FEAT-17.SPEC-001 (Dinner Vote Casting, vote-after-resolution rejected); FEAT-23.SPEC-002 (Pick/Change a Recipe, reject-with-refresh); dependency-map Contention lines on 16 of 17 entities document non-"None" concurrency-resolution rules |
| Compliance/privacy | Yes | FEAT-01.SPEC-007 (Parental Consent Confirmation -- required before a kid profile is created); FEAT-01.SPEC-014 and FEAT-01.SPEC-016 (kid-profile data-minimality and access restrictions); FEAT-18.SPEC-001 (Export Household Data -- data-subject export right); FEAT-18.SPEC-003 (Delete Household); FEAT-22.SPEC-002 (Support Read-Only Household View, kid-data visibility restricted); FEAT-22.SPEC-007 (Kid Profile & Billing Data Visibility Rule); dependency-map Data Sensitivity lines note "children's-privacy-class protection" for Member Profile, Dietary Rule, and Dinner Vote and "GDPR-class handling" elsewhere; ASMP-26 (assumptions-constraints.md, ## Non-Functional Expectations -- "children's data is minimal, parent-controlled... no data is ever sold or used for advertising"); ASMP-27 ("Compliance: because the product processes children's data in the US and UK, it is treated as subject to children's-privacy-class protections... alongside general personal-data protection for all household members, including the right to a copy of their data and to deletion; no medical-data regime applies") |
| Internationalization | Yes | FEAT-16.SPEC-001 (Units & Currency Settings -- cups/oz vs grams/ml, currency from a supported set covering at least USD and GBP); FEAT-16.SPEC-002 (Aisle Name Customization); FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule); BRIEF.md, ## Scale & Non-Functional Expectations ("Geography: the US and UK first. Units (cups vs grams), currency and supermarket aisle names must be configurable, not hard-coded"); ASMP-28 (assumptions-constraints.md, ## Non-Functional Expectations -- "Localization: measurement units, currency, and supermarket aisle names are configurable per household to support both US and UK households from launch; the product is in English only") |
| Scale hints | Yes | ASMP-23 (assumptions-constraints.md, ## Non-Functional Expectations -- "a weekly plan generates within well under a minute with an explained wait, and recipe search results appear within about a second"); ASMP-24 ("several thousand households in the first year, 2-6 members each, with the product staying equally responsive as that base grows; every household's history is kept for the life of the account"); ASMP-22 ("the shared grocery list feels instant when items are ticked or added, and a meal swap completes and reflects in the list within seconds"); BRIEF.md, ## Scale & Non-Functional Expectations ("Order of magnitude: several thousand households in the first year, with 2-6 people each") |

## 4. Derived Classifications

**Classification rule:** the classification describes blueprint scope -- how much product definition the architecture must cover -- not deployment scale. Deployment-scale evidence (users, traffic, data growth) lives in Section 7 and must never be derived from document counts.

### Threshold Definitions

| Axis | Small | Medium | Large |
|------|-------|--------|-------|
| Scale | 1-8 features, <=40 specs | 9-24 features, 41-200 specs | 25+ features, 201+ specs |
| Data Complexity | <=10 entities, few cross-entity rules | 11-30 entities, moderate relationship graph and cross-entity rules | 31+ entities, dense relationship graph |
| Interaction Complexity | Mostly CRUD screens, <=5 automations, no real-time or collaboration signals | Mixed screens with state, 6-15 automations, integration or notification behavior present | Complex state machines, 16+ automations, real-time or collaboration signals present |

### Project Classification

| Axis | Classification | Evidence |
|------|---------------|----------|
| Scale | Medium | 25 features, 196 specs (falls at the Medium-band upper edge: 25 features is the first value of the Large band's 25+ feature threshold, but 196 specs is within the Medium band's 41-200 spec range; the spec count, the larger and more granular of the two counts, controls) |
| Data Complexity | Medium | 17 entities (within the 11-30 Medium band), 54 inter-entity relationships and 20 cross-feature business rules (XBR) -- a moderate, densely cross-referenced relationship graph rather than a sparse one |
| Interaction Complexity | Large | 46 Automation specs (exceeds the 16+ Large-band threshold); real-time signals present (FEAT-03.SPEC-011, FEAT-06.SPEC-005); collaboration/concurrency signals present across 16 of 17 entities' Contention notes |

**Summary:** Medium / Medium / Large

(Blueprint scope only -- the deployment-scale evidence for this product lives in Section 7.)

## 5. Entity Inventory

Complete listing of all entities from the Domain Entity Inventory in product-features.md, enriched with the Shared Data Entities detail from feature-dependency-map.md where an entity appears there.

| Entity Name | Managing Feature | Field Count | Relationships |
|-------------|-----------------|-------------|---------------|
| Household | FEAT-01 (Household Setup & Member Profiles), FEAT-16 (Units, Currency & Locale Configuration), FEAT-14 (Subscription & Billing Management), FEAT-18 (Account & Data Management) | 9 | Has 1-12 Member Profiles (one of them the organiser), one Subscription, one Weekly Plan per week, one Grocery List per week, many Pantry Items, many imported Recipes, and many Support Requests; may be the referring or referred household of a Household Referral. One household per account in v1 (ASMP-16). |
| Member Profile | FEAT-01 (Household Setup & Member Profiles), FEAT-09 (Household Invitations & Membership) (leaving, organiser hand-over), FEAT-18 (Account & Data Management) (removal, own-account management) | 7 | Belongs to one Household; owns zero or more Dietary Rules; authors Ratings, Swap Suggestions, Support Requests, and (older kids, Later) Dinner Votes; an adult member owns one personal household-referral link. |
| Dietary Rule | FEAT-01 (Household Setup & Member Profiles), FEAT-12 (Meal Rating & Preference Learning) (for learned dislikes) | 6 | Belongs to one Member Profile; read by the safety engine against every ingredient of every candidate Recipe. |
| Weekly Plan | FEAT-03 (AI Weekly Dinner Plan Generation), FEAT-23 (Manual Weekly Planning), FEAT-04 (One-Tap Meal Swap) | 6 | Belongs to one Household; contains up to seven dinner Planned Meals plus leftover-lunch Planned Meals; drives one Grocery List; receives Swap Suggestions and (Later) Dinner Votes. |
| Planned Meal | FEAT-04 (One-Tap Meal Swap), FEAT-23 (Manual Weekly Planning), FEAT-02 (Dietary Rules & Allergy Safety Engine) (removal after a safety concern or a mid-week rule change) | 9 | Belongs to one Weekly Plan; references one Recipe; contributes ingredients to the Grocery List; is the subject of Ratings, Swap Suggestions, a Tonight's Dinner nudge, and (Later) Dinner Votes and calendar entries. |
| Recipe | FEAT-08 (Recipe Library (Starter Recipes)) (starter recipes are read-only for households), FEAT-10 (Recipe Import from Web Link) (households edit or remove their own imported recipes) | 9 | Referenced by Planned Meals; imported recipes belong to one Household; starter recipes are shared read-only content for all households. |
| Pantry Item | FEAT-05 (Pantry-Aware Suggestions) | 4 | Belongs to one Household; matched against Planned Meal ingredients for pantry callouts and against Grocery List lines to leave them off. |
| Grocery List | FEAT-06 (Shared Grocery List) | 3 | Belongs to one Household and one Weekly Plan; contains Grocery List Items. |
| Grocery List Item | FEAT-06 (Shared Grocery List) | 5 | Belongs to one Grocery List; plan-derived items trace to one or more Planned Meals. |
| Rating | FEAT-12 (Meal Rating & Preference Learning) | 4 | Links one Member Profile to one Planned Meal; repeated down-ratings feed a learned soft dislike in Dietary Rule. |
| Invitation | FEAT-09 (Household Invitations & Membership) | 3 | Belongs to one Household; on acceptance creates one Member Profile with Other Adult Member access; triggers one Member Onboarding. |
| Subscription | FEAT-14 (Subscription & Billing Management) | 4 | Belongs to one Household; payment details are held with the payment-processing capability and seen only by the organiser. |
| Swap Suggestion | FEAT-04 (One-Tap Meal Swap) | 4 | Targets one Planned Meal slot in one Weekly Plan; raised by one Member Profile. |
| Dinner Vote | FEAT-17 (Older-Kid Dinner Voting) | 4 | Belongs to one Weekly Plan night; cast by an older-kid Member Profile. |
| Household Referral | FEAT-24 (Invite Another Household) | 5 | Links two Households; a new household is attributed to at most one referring household and cannot refer itself. |
| Waste & Spend Check-In | FEAT-25 (Weekly Waste & Spend Check-In) | -- | -- |
| Support Request | FEAT-22 (Operator Read-Only Support Access) (status only -- it never changes household data) | 6 | Belongs to one Household; a safety concern references one Planned Meal and its Recipe; it is the precondition for any operator support view. |

## 6. Raw Spec Index

Complete listing of all 196 specs across all 25 features, compiled from all feature-overview.md Spec Inventory tables in FEAT-NN then SPEC-NNN ascending order. Key Signals cross-reference the Section 3 results.

| Spec ID | Name | Type | Feature | Key Signals |
|---------|------|------|---------|-------------|
| FEAT-01.SPEC-001 | Account Sign-Up & Sign-In | Screen | FEAT-01 (Household Setup & Member Profiles) | Authentication (role-based) |
| FEAT-01.SPEC-002 | Password Recovery | Screen | FEAT-01 (Household Setup & Member Profiles) | Authentication (role-based) |
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | Screen | FEAT-01 (Household Setup & Member Profiles) | Collaboration/concurrency |
| FEAT-01.SPEC-004 | Member List & Add Member | Screen | FEAT-01 (Household Setup & Member Profiles) | Real-time, Collaboration/concurrency |
| FEAT-01.SPEC-005 | Member Profile Detail | Screen | FEAT-01 (Household Setup & Member Profiles) | Collaboration/concurrency |
| FEAT-01.SPEC-006 | Dietary Rules Editor | Screen | FEAT-01 (Household Setup & Member Profiles) | Collaboration/concurrency |
| FEAT-01.SPEC-007 | Parental Consent Confirmation | Screen | FEAT-01 (Household Setup & Member Profiles) | Compliance/privacy |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | Screen | FEAT-01 (Household Setup & Member Profiles) | Collaboration/concurrency |
| FEAT-01.SPEC-009 | Setup Complete & Next Steps | Screen | FEAT-01 (Household Setup & Member Profiles) | -- |
| FEAT-01.SPEC-010 | Household Settings Hub | Screen | FEAT-01 (Household Setup & Member Profiles) | -- |
| FEAT-01.SPEC-011 | Default Subscription Provisioning | Automation | FEAT-01 (Household Setup & Member Profiles) | Payments/billing |
| FEAT-01.SPEC-012 | Mid-Week Hard-Rule Change Trigger | Automation | FEAT-01 (Household Setup & Member Profiles) | -- |
| FEAT-01.SPEC-013 | Setup Draft Persistence & Offline Queuing | Logic/Rule | FEAT-01 (Household Setup & Member Profiles) | Offline |
| FEAT-01.SPEC-014 | Household & Member Field Validation Rules | Logic/Rule | FEAT-01 (Household Setup & Member Profiles) | Compliance/privacy |
| FEAT-01.SPEC-015 | Dietary Rule Classification & Allergen Matching Rules | Logic/Rule | FEAT-01 (Household Setup & Member Profiles) | -- |
| FEAT-01.SPEC-016 | Household Setup Authorization Rules | Logic/Rule | FEAT-01 (Household Setup & Member Profiles) | Authentication (role-based), Compliance/privacy |
| FEAT-01.SPEC-017 | Transactional Email Integration (Account & Recovery) | Integration | FEAT-01 (Household Setup & Member Profiles) | Third-party integrations |
| FEAT-01.SPEC-018 | Mid-Week Rule Change Notification | Notification | FEAT-01 (Household Setup & Member Profiles) | Notifications (email/push/SMS) |
| FEAT-02.SPEC-001 | Report a Safety Concern | Screen | FEAT-02 (Dietary Rules & Allergy Safety Engine) | -- |
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | Automation | FEAT-02 (Dietary Rules & Allergy Safety Engine) | -- |
| FEAT-02.SPEC-003 | Mid-Week Rule Change Re-Check | Automation | FEAT-02 (Dietary Rules & Allergy Safety Engine) | -- |
| FEAT-02.SPEC-004 | Safety Concern Intake & Removal | Automation | FEAT-02 (Dietary Rules & Allergy Safety Engine) | -- |
| FEAT-02.SPEC-005 | Safety Concern Resolution Outcome | Automation | FEAT-02 (Dietary Rules & Allergy Safety Engine) | -- |
| FEAT-02.SPEC-006 | Rule Strength & Blocking Policy | Logic/Rule | FEAT-02 (Dietary Rules & Allergy Safety Engine) | -- |
| FEAT-02.SPEC-007 | Ingredient Data Completeness & Fail-Closed Policy | Logic/Rule | FEAT-02 (Dietary Rules & Allergy Safety Engine) | -- |
| FEAT-02.SPEC-008 | Safety Badge & Disclaimer Display Rule | Logic/Rule | FEAT-02 (Dietary Rules & Allergy Safety Engine) | -- |
| FEAT-02.SPEC-009 | Safety Concern Eligibility & Re-offer Policy | Logic/Rule | FEAT-02 (Dietary Rules & Allergy Safety Engine) | -- |
| FEAT-02.SPEC-010 | Transactional Email Delivery (Safety Reports) | Integration | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Third-party integrations |
| FEAT-02.SPEC-011 | Safety Concern Reporter Acknowledgement | Notification | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Notifications (email/push/SMS) |
| FEAT-02.SPEC-012 | Safety Concern Organiser Alert | Notification | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Notifications (email/push/SMS) |
| FEAT-02.SPEC-013 | Safety Concern Operator Alert | Notification | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Notifications (email/push/SMS) |
| FEAT-02.SPEC-014 | Safety Concern Resolution Notice | Notification | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Notifications (email/push/SMS) |
| FEAT-03.SPEC-001 | Weekly Plan View | Screen | FEAT-03 (AI Weekly Dinner Plan Generation) | -- |
| FEAT-03.SPEC-002 | Free-Tier Plan Placeholder & Upgrade Prompt | Screen | FEAT-03 (AI Weekly Dinner Plan Generation) | -- |
| FEAT-03.SPEC-003 | Scheduled Weekly Plan Generation | Automation | FEAT-03 (AI Weekly Dinner Plan Generation) | Background processing |
| FEAT-03.SPEC-004 | First-Plan Generation on Upgrade | Automation | FEAT-03 (AI Weekly Dinner Plan Generation) | -- |
| FEAT-03.SPEC-005 | Auto-Adoption at Week Start | Automation | FEAT-03 (AI Weekly Dinner Plan Generation) | -- |
| FEAT-03.SPEC-006 | Budget Fit & Estimated Total Rule | Logic/Rule | FEAT-03 (AI Weekly Dinner Plan Generation) | -- |
| FEAT-03.SPEC-007 | Household-Scaled Quantity Rule | Logic/Rule | FEAT-03 (AI Weekly Dinner Plan Generation) | -- |
| FEAT-03.SPEC-008 | Plan Approval Authorization Rule | Logic/Rule | FEAT-03 (AI Weekly Dinner Plan Generation) | -- |
| FEAT-03.SPEC-009 | Generation Eligibility & Tier-Gating Rule | Logic/Rule | FEAT-03 (AI Weekly Dinner Plan Generation) | -- |
| FEAT-03.SPEC-010 | AI Plan Generation Capability Integration | Integration | FEAT-03 (AI Weekly Dinner Plan Generation) | Third-party integrations, AI/ML behavior |
| FEAT-03.SPEC-011 | Real-Time Plan Sync Integration | Integration | FEAT-03 (AI Weekly Dinner Plan Generation) | Real-time, Third-party integrations, Collaboration/concurrency |
| FEAT-04.SPEC-001 | Meal Swap (Direct) | Screen | FEAT-04 (One-Tap Meal Swap) | -- |
| FEAT-04.SPEC-002 | Suggest a Swap | Screen | FEAT-04 (One-Tap Meal Swap) | -- |
| FEAT-04.SPEC-003 | Review Swap Suggestions | Screen | FEAT-04 (One-Tap Meal Swap) | -- |
| FEAT-04.SPEC-004 | Apply Meal Swap | Automation | FEAT-04 (One-Tap Meal Swap) | Collaboration/concurrency |
| FEAT-04.SPEC-005 | Suggestion Lapse | Automation | FEAT-04 (One-Tap Meal Swap) | Background processing |
| FEAT-04.SPEC-006 | Swap Suggestion Notifications | Notification | FEAT-04 (One-Tap Meal Swap) | Notifications (email/push/SMS) |
| FEAT-04.SPEC-007 | Swap Alternatives Generation | Integration | FEAT-04 (One-Tap Meal Swap) | Third-party integrations, AI/ML behavior |
| FEAT-04.SPEC-008 | Alternatives Computation & Scarcity Explanation | Logic/Rule | FEAT-04 (One-Tap Meal Swap) | AI/ML behavior |
| FEAT-04.SPEC-009 | Swap Concurrency Lock | Logic/Rule | FEAT-04 (One-Tap Meal Swap) | -- |
| FEAT-04.SPEC-010 | Suggestion Lifecycle Rules | Logic/Rule | FEAT-04 (One-Tap Meal Swap) | -- |
| FEAT-05.SPEC-001 | Pantry List & Item Entry | Screen | FEAT-05 (Pantry-Aware Suggestions) | -- |
| FEAT-05.SPEC-002 | Used-It-Up Prompt Trigger | Automation | FEAT-05 (Pantry-Aware Suggestions) | -- |
| FEAT-05.SPEC-003 | Pantry Item Duplicate Merge | Automation | FEAT-05 (Pantry-Aware Suggestions) | -- |
| FEAT-05.SPEC-004 | Pantry Item Field Validation | Logic/Rule | FEAT-05 (Pantry-Aware Suggestions) | -- |
| FEAT-05.SPEC-005 | Pantry-Aware Plan Weighting Tier Gate | Logic/Rule | FEAT-05 (Pantry-Aware Suggestions) | -- |
| FEAT-05.SPEC-006 | Pantry-to-Recipe Matching for Plan Callout | Logic/Rule | FEAT-05 (Pantry-Aware Suggestions) | -- |
| FEAT-05.SPEC-007 | Pantry Item Off-Grocery-List Exclusion Rule | Logic/Rule | FEAT-05 (Pantry-Aware Suggestions) | -- |
| FEAT-06.SPEC-001 | Grocery List | Screen | FEAT-06 (Shared Grocery List) | -- |
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Automation | FEAT-06 (Shared Grocery List) | -- |
| FEAT-06.SPEC-003 | "Already Have It" Handling | Automation | FEAT-06 (Shared Grocery List) | -- |
| FEAT-06.SPEC-004 | Week Rollover & Carryover | Automation | FEAT-06 (Shared Grocery List) | Background processing |
| FEAT-06.SPEC-005 | Live Grocery List Sync | Integration | FEAT-06 (Shared Grocery List) | Real-time, Third-party integrations, Collaboration/concurrency |
| FEAT-06.SPEC-006 | Ingredient Consolidation & Quantity Derivation | Logic/Rule | FEAT-06 (Shared Grocery List) | -- |
| FEAT-06.SPEC-007 | Manual Item Validation & Duplicate Merge | Logic/Rule | FEAT-06 (Shared Grocery List) | Collaboration/concurrency |
| FEAT-06.SPEC-008 | Offline Conflict Resolution Rules | Logic/Rule | FEAT-06 (Shared Grocery List) | Offline, Collaboration/concurrency |
| FEAT-06.SPEC-009 | Grocery List Access & Authorization Rules | Logic/Rule | FEAT-06 (Shared Grocery List) | -- |
| FEAT-07.SPEC-001 | Plan-Ready Notification Trigger | Automation | FEAT-07 (Weekly Plan Ready Notification) | Background processing |
| FEAT-07.SPEC-002 | Plan-Ready Notification Message | Notification | FEAT-07 (Weekly Plan Ready Notification) | Notifications (email/push/SMS) |
| FEAT-07.SPEC-003 | Plan-Ready Delivery & Eligibility Rules | Logic/Rule | FEAT-07 (Weekly Plan Ready Notification) | -- |
| FEAT-07.SPEC-004 | Plan-Arrival Day & Time Setting Rule | Logic/Rule | FEAT-07 (Weekly Plan Ready Notification) | -- |
| FEAT-07.SPEC-005 | Device-Notification Delivery Integration | Integration | FEAT-07 (Weekly Plan Ready Notification) | Third-party integrations |
| FEAT-07.SPEC-006 | Plan-Ready Email Fallback Integration | Integration | FEAT-07 (Weekly Plan Ready Notification) | Third-party integrations |
| FEAT-08.SPEC-001 | Recipe Library Browse & Search | Screen | FEAT-08 (Recipe Library (Starter Recipes)) | Search (simple filter) |
| FEAT-08.SPEC-002 | Recipe Detail View | Screen | FEAT-08 (Recipe Library (Starter Recipes)) | -- |
| FEAT-08.SPEC-003 | Ineligible Recipe Search Disclosure Rule | Logic/Rule | FEAT-08 (Recipe Library (Starter Recipes)) | -- |
| FEAT-08.SPEC-004 | Starter Recipe Content Seeding & Maintenance | Integration | FEAT-08 (Recipe Library (Starter Recipes)) | Third-party integrations |
| FEAT-09.SPEC-001 | Household Invitations Manager | Screen | FEAT-09 (Household Invitations & Membership) | -- |
| FEAT-09.SPEC-002 | Invitation Acceptance | Screen | FEAT-09 (Household Invitations & Membership) | -- |
| FEAT-09.SPEC-003 | Organiser Hand-Over Initiation | Screen | FEAT-09 (Household Invitations & Membership) | Collaboration/concurrency |
| FEAT-09.SPEC-004 | Organiser Hand-Over Acceptance | Screen | FEAT-09 (Household Invitations & Membership) | Collaboration/concurrency |
| FEAT-09.SPEC-005 | Leave Household | Screen | FEAT-09 (Household Invitations & Membership) | -- |
| FEAT-09.SPEC-006 | Invitation Expiry | Automation | FEAT-09 (Household Invitations & Membership) | Background processing |
| FEAT-09.SPEC-007 | Invitation Acceptance Processing | Automation | FEAT-09 (Household Invitations & Membership) | -- |
| FEAT-09.SPEC-008 | Member Departure Processing | Automation | FEAT-09 (Household Invitations & Membership) | -- |
| FEAT-09.SPEC-009 | Organiser Hand-Over Processing | Automation | FEAT-09 (Household Invitations & Membership) | -- |
| FEAT-09.SPEC-010 | Invitation & Membership Validation Rules | Logic/Rule | FEAT-09 (Household Invitations & Membership) | -- |
| FEAT-09.SPEC-011 | Household Invitations & Membership Authorization Rules | Logic/Rule | FEAT-09 (Household Invitations & Membership) | -- |
| FEAT-09.SPEC-012 | Invitation Accepted Confirmation | Notification | FEAT-09 (Household Invitations & Membership) | Notifications (email/push/SMS) |
| FEAT-09.SPEC-013 | Member Left Household Notification | Notification | FEAT-09 (Household Invitations & Membership) | Notifications (email/push/SMS) |
| FEAT-09.SPEC-014 | Organiser Hand-Over Request Notification | Notification | FEAT-09 (Household Invitations & Membership) | Notifications (email/push/SMS) |
| FEAT-10.SPEC-001 | Import by Link | Screen | FEAT-10 (Recipe Import from Web Link) | Import/export |
| FEAT-10.SPEC-002 | Review Extracted Recipe | Screen | FEAT-10 (Recipe Import from Web Link) | -- |
| FEAT-10.SPEC-003 | Manual Recipe Entry | Screen | FEAT-10 (Recipe Import from Web Link) | -- |
| FEAT-10.SPEC-004 | Edit Imported Recipe | Screen | FEAT-10 (Recipe Import from Web Link) | Collaboration/concurrency |
| FEAT-10.SPEC-005 | Web Page Recipe Extraction | Integration | FEAT-10 (Recipe Import from Web Link) | Third-party integrations |
| FEAT-10.SPEC-006 | Duplicate Import Detection | Automation | FEAT-10 (Recipe Import from Web Link) | -- |
| FEAT-10.SPEC-007 | Recipe Import Validation & Rate Limit Rules | Logic/Rule | FEAT-10 (Recipe Import from Web Link) | -- |
| FEAT-10.SPEC-008 | Safety Re-check on Import Save/Edit | Automation | FEAT-10 (Recipe Import from Web Link) | -- |
| FEAT-11.SPEC-001 | Leftover Lunch Card | Screen | FEAT-11 (Leftover Rollover to Lunches) | -- |
| FEAT-11.SPEC-002 | Leftover Lunch Suggestion Generation | Automation | FEAT-11 (Leftover Rollover to Lunches) | -- |
| FEAT-11.SPEC-003 | Leftover Lunch Eligibility & Linking Rule | Logic/Rule | FEAT-11 (Leftover Rollover to Lunches) | -- |
| FEAT-11.SPEC-004 | Leftover Lunch Withdrawal on Source Change | Automation | FEAT-11 (Leftover Rollover to Lunches) | -- |
| FEAT-12.SPEC-001 | Post-Dinner Rating Prompt | Screen | FEAT-12 (Meal Rating & Preference Learning) | -- |
| FEAT-12.SPEC-002 | Rating Submission, Change & Proxy Rules | Logic/Rule | FEAT-12 (Meal Rating & Preference Learning) | Collaboration/concurrency |
| FEAT-12.SPEC-003 | Rating Access & Authorization Rules | Logic/Rule | FEAT-12 (Meal Rating & Preference Learning) | -- |
| FEAT-12.SPEC-004 | Preference Weighting & Tier-Gating Rule | Logic/Rule | FEAT-12 (Meal Rating & Preference Learning) | AI/ML behavior |
| FEAT-12.SPEC-005 | Repeated-Dislike Learned Update | Automation | FEAT-12 (Meal Rating & Preference Learning) | AI/ML behavior |
| FEAT-13.SPEC-001 | Tonight's Nudge Trigger | Automation | FEAT-13 (Tonight's Dinner Reminder) | Background processing |
| FEAT-13.SPEC-002 | Tonight's Dinner Nudge Message | Notification | FEAT-13 (Tonight's Dinner Reminder) | Notifications (email/push/SMS) |
| FEAT-13.SPEC-003 | Same-Day Swap Correction Trigger | Automation | FEAT-13 (Tonight's Dinner Reminder) | -- |
| FEAT-13.SPEC-004 | Same-Day Swap Correction Message | Notification | FEAT-13 (Tonight's Dinner Reminder) | Notifications (email/push/SMS) |
| FEAT-13.SPEC-005 | Nudge Delivery & Eligibility Rules | Logic/Rule | FEAT-13 (Tonight's Dinner Reminder) | -- |
| FEAT-13.SPEC-006 | Prep-Reminder Derivation Rule | Logic/Rule | FEAT-13 (Tonight's Dinner Reminder) | -- |
| FEAT-14.SPEC-001 | Plan Tier Overview | Screen | FEAT-14 (Subscription & Billing Management) | Payments/billing |
| FEAT-14.SPEC-002 | Upgrade to Paid | Screen | FEAT-14 (Subscription & Billing Management) | Payments/billing |
| FEAT-14.SPEC-003 | Billing & Payment Management | Screen | FEAT-14 (Subscription & Billing Management) | Payments/billing, Collaboration/concurrency |
| FEAT-14.SPEC-004 | Downgrade / Cancel | Screen | FEAT-14 (Subscription & Billing Management) | Payments/billing |
| FEAT-14.SPEC-005 | Billing State & Refund Rules | Logic/Rule | FEAT-14 (Subscription & Billing Management) | Payments/billing |
| FEAT-14.SPEC-006 | Tier & Billing Access Authorization | Logic/Rule | FEAT-14 (Subscription & Billing Management) | Payments/billing |
| FEAT-14.SPEC-007 | Payment Failure & Grace Period Handling | Automation | FEAT-14 (Subscription & Billing Management) | Payments/billing |
| FEAT-14.SPEC-008 | Apply Subscription Change | Automation | FEAT-14 (Subscription & Billing Management) | Payments/billing |
| FEAT-14.SPEC-009 | Payment Processing Integration | Integration | FEAT-14 (Subscription & Billing Management) | Payments/billing, Third-party integrations |
| FEAT-14.SPEC-010 | Billing Confirmation Notification | Notification | FEAT-14 (Subscription & Billing Management) | Payments/billing, Notifications (email/push/SMS) |
| FEAT-14.SPEC-011 | Payment Failure Grace-Period Notice | Notification | FEAT-14 (Subscription & Billing Management) | Payments/billing, Notifications (email/push/SMS) |
| FEAT-14.SPEC-012 | Transactional Email Delivery (Billing) | Integration | FEAT-14 (Subscription & Billing Management) | Payments/billing, Third-party integrations |
| FEAT-15.SPEC-001 | Onboarding Landing | Screen | FEAT-15 (Member Onboarding) | -- |
| FEAT-15.SPEC-002 | Onboarding Trigger & Completion | Automation | FEAT-15 (Member Onboarding) | -- |
| FEAT-15.SPEC-003 | Onboarding Eligibility & Once-Only Rule | Logic/Rule | FEAT-15 (Member Onboarding) | -- |
| FEAT-16.SPEC-001 | Units & Currency Settings | Screen | FEAT-16 (Units, Currency & Locale Configuration) | Internationalization |
| FEAT-16.SPEC-002 | Aisle Name Customization | Screen | FEAT-16 (Units, Currency & Locale Configuration) | Internationalization |
| FEAT-16.SPEC-003 | Locale Configuration Validation & Defaults | Logic/Rule | FEAT-16 (Units, Currency & Locale Configuration) | -- |
| FEAT-16.SPEC-004 | Cross-Feature Value Conversion Rule | Logic/Rule | FEAT-16 (Units, Currency & Locale Configuration) | Internationalization |
| FEAT-17.SPEC-001 | Dinner Vote Casting | Screen | FEAT-17 (Older-Kid Dinner Voting) | Collaboration/concurrency |
| FEAT-17.SPEC-002 | Voting Round Setup | Screen | FEAT-17 (Older-Kid Dinner Voting) | -- |
| FEAT-17.SPEC-003 | Vote Outcome & Resolution | Screen | FEAT-17 (Older-Kid Dinner Voting) | -- |
| FEAT-17.SPEC-004 | Voting Round Creation & Safety Validation | Automation | FEAT-17 (Older-Kid Dinner Voting) | -- |
| FEAT-17.SPEC-005 | Vote Round Resolution & Fallback | Automation | FEAT-17 (Older-Kid Dinner Voting) | -- |
| FEAT-17.SPEC-006 | Dinner Voting Rules -- Access, Validation & Conflict Resolution | Logic/Rule | FEAT-17 (Older-Kid Dinner Voting) | -- |
| FEAT-18.SPEC-001 | Export Household Data | Screen | FEAT-18 (Account & Data Management) | Import/export, Compliance/privacy |
| FEAT-18.SPEC-002 | Remove Member Profile | Screen | FEAT-18 (Account & Data Management) | -- |
| FEAT-18.SPEC-003 | Delete Household | Screen | FEAT-18 (Account & Data Management) | Compliance/privacy |
| FEAT-18.SPEC-004 | My Account | Screen | FEAT-18 (Account & Data Management) | -- |
| FEAT-18.SPEC-005 | Contact Support | Screen | FEAT-18 (Account & Data Management) | -- |
| FEAT-18.SPEC-006 | Export Generation Processing | Automation | FEAT-18 (Account & Data Management) | Import/export |
| FEAT-18.SPEC-007 | Member Removal Processing | Automation | FEAT-18 (Account & Data Management) | -- |
| FEAT-18.SPEC-008 | Household Deletion Processing | Automation | FEAT-18 (Account & Data Management) | -- |
| FEAT-18.SPEC-009 | Own Account Deletion Processing | Automation | FEAT-18 (Account & Data Management) | -- |
| FEAT-18.SPEC-010 | Account & Data Validation Rules | Logic/Rule | FEAT-18 (Account & Data Management) | -- |
| FEAT-18.SPEC-011 | Account & Data Authorization Rules | Logic/Rule | FEAT-18 (Account & Data Management) | Authentication (role-based) |
| FEAT-18.SPEC-012 | Transactional Email (Account & Data) | Integration | FEAT-18 (Account & Data Management) | Third-party integrations |
| FEAT-18.SPEC-013 | Export Ready Notification | Notification | FEAT-18 (Account & Data Management) | Notifications (email/push/SMS) |
| FEAT-18.SPEC-014 | Household Deletion Completed Notification | Notification | FEAT-18 (Account & Data Management) | Notifications (email/push/SMS) |
| FEAT-18.SPEC-015 | Support Request Acknowledgement | Notification | FEAT-18 (Account & Data Management) | Notifications (email/push/SMS) |
| FEAT-19.SPEC-001 | Weekly Plan History Browse | Screen | FEAT-19 (Weekly Plan History) | -- |
| FEAT-19.SPEC-002 | Past Week Detail View | Screen | FEAT-19 (Weekly Plan History) | -- |
| FEAT-19.SPEC-003 | Past Plan Reuse | Automation | FEAT-19 (Weekly Plan History) | -- |
| FEAT-19.SPEC-004 | History Access & Reuse Authorization | Logic/Rule | FEAT-19 (Weekly Plan History) | -- |
| FEAT-20.SPEC-001 | Grocery Handoff Screen | Screen | FEAT-20 (Online Grocery Ordering Handoff) | -- |
| FEAT-20.SPEC-002 | Online Grocery Ordering Integration | Integration | FEAT-20 (Online Grocery Ordering Handoff) | Third-party integrations |
| FEAT-20.SPEC-003 | Handoff Eligibility & Authorization Rules | Logic/Rule | FEAT-20 (Online Grocery Ordering Handoff) | -- |
| FEAT-21.SPEC-001 | Calendar Connection Settings | Screen | FEAT-21 (Family Calendar Sync) | -- |
| FEAT-21.SPEC-002 | Family Calendar Integration | Integration | FEAT-21 (Family Calendar Sync) | Third-party integrations |
| FEAT-21.SPEC-003 | Weekly Dinner Calendar Sync | Automation | FEAT-21 (Family Calendar Sync) | -- |
| FEAT-21.SPEC-004 | Calendar Connection & Sync Governance Rules | Logic/Rule | FEAT-21 (Family Calendar Sync) | -- |
| FEAT-21.SPEC-005 | Calendar Entry Content Derivation Rule | Logic/Rule | FEAT-21 (Family Calendar Sync) | -- |
| FEAT-22.SPEC-001 | Support Request Queue | Screen | FEAT-22 (Operator Read-Only Support Access) | -- |
| FEAT-22.SPEC-002 | Support Read-Only Household View | Screen | FEAT-22 (Operator Read-Only Support Access) | Compliance/privacy |
| FEAT-22.SPEC-003 | Household Support Access Record | Screen | FEAT-22 (Operator Read-Only Support Access) | -- |
| FEAT-22.SPEC-004 | Support Access Session Logging | Automation | FEAT-22 (Operator Read-Only Support Access) | -- |
| FEAT-22.SPEC-005 | Support Request Resolution | Automation | FEAT-22 (Operator Read-Only Support Access) | -- |
| FEAT-22.SPEC-006 | Support Access Scope & Gating Rules | Logic/Rule | FEAT-22 (Operator Read-Only Support Access) | -- |
| FEAT-22.SPEC-007 | Kid Profile & Billing Data Visibility Rule | Logic/Rule | FEAT-22 (Operator Read-Only Support Access) | Compliance/privacy |
| FEAT-22.SPEC-008 | Support Request Status Transition Rules | Logic/Rule | FEAT-22 (Operator Read-Only Support Access) | -- |
| FEAT-22.SPEC-009 | Support View Recorded Notification | Notification | FEAT-22 (Operator Read-Only Support Access) | Notifications (email/push/SMS) |
| FEAT-22.SPEC-010 | Support Request Resolved Notification | Notification | FEAT-22 (Operator Read-Only Support Access) | Notifications (email/push/SMS) |
| FEAT-23.SPEC-001 | Weekly Plan (Manual Week Builder) | Screen | FEAT-23 (Manual Weekly Planning) | -- |
| FEAT-23.SPEC-002 | Pick / Change a Recipe | Screen | FEAT-23 (Manual Weekly Planning) | Collaboration/concurrency |
| FEAT-23.SPEC-003 | Suggest a Pick | Screen | FEAT-23 (Manual Weekly Planning) | -- |
| FEAT-23.SPEC-004 | Safe-Choice Filtering & Placement Block | Logic/Rule | FEAT-23 (Manual Weekly Planning) | -- |
| FEAT-23.SPEC-005 | Manual Planning Validation & Limits | Logic/Rule | FEAT-23 (Manual Weekly Planning) | -- |
| FEAT-23.SPEC-006 | Apply Manual Pick | Automation | FEAT-23 (Manual Weekly Planning) | -- |
| FEAT-24.SPEC-001 | Invite Another Household Screen | Screen | FEAT-24 (Invite Another Household) | -- |
| FEAT-24.SPEC-002 | Referral Welcome Screen | Screen | FEAT-24 (Invite Another Household) | -- |
| FEAT-24.SPEC-003 | Personal Referral Link Provisioning | Automation | FEAT-24 (Invite Another Household) | -- |
| FEAT-24.SPEC-004 | Household Referral Recording | Automation | FEAT-24 (Invite Another Household) | -- |
| FEAT-24.SPEC-005 | Referral Upgrade Tracking | Automation | FEAT-24 (Invite Another Household) | -- |
| FEAT-24.SPEC-006 | Household Referral Rules | Logic/Rule | FEAT-24 (Invite Another Household) | -- |
| FEAT-24.SPEC-007 | Referral Joined Notification | Notification | FEAT-24 (Invite Another Household) | Notifications (email/push/SMS) |
| FEAT-25.SPEC-001 | Weekly Check-In Card | Screen | FEAT-25 (Weekly Waste & Spend Check-In) | -- |
| FEAT-25.SPEC-002 | Check-In Trend View | Screen | FEAT-25 (Weekly Waste & Spend Check-In) | -- |
| FEAT-25.SPEC-003 | Weekly Check-In Cycle | Automation | FEAT-25 (Weekly Waste & Spend Check-In) | Background processing |
| FEAT-25.SPEC-004 | Check-In Trend Calculation | Logic/Rule | FEAT-25 (Weekly Waste & Spend Check-In) | -- |
| FEAT-25.SPEC-005 | Check-In Validation & Access Rules | Logic/Rule | FEAT-25 (Weekly Waste & Spend Check-In) | -- |

## 7. Demand-Side Inputs

### From BRIEF.md -- Scale & Non-Functional Expectations

> "- **Order of magnitude:** several thousand households in the first year, with 2-6 people each.
> - **Devices / platforms:** a mobile-first responsive web app. People plan on the sofa and shop with the phone in one hand, and the organiser may use a laptop for setup. There are no native apps in v1.
> - **Geography:** the US and UK first. Units (cups vs grams), currency and supermarket aisle names must be configurable, not hard-coded.
> - **Performance / availability:** the shared grocery list must feel instant and must keep working in a supermarket with bad signal. Changes made offline sync later. No specific uptime number was stated.
> - **Cost:** the AI cost per household must stay small, roughly one weekly plan plus a few swaps. The free tier gets no AI.
> - **Privacy:** children's data is minimal, parent-controlled and never used for anything but the family's own plan. There are no ads and no data selling."
> -- BRIEF.md, ## Scale & Non-Functional Expectations

### From BRIEF.md -- Ecosystem & Integrations

> "- **AI model:** generates the weekly plans and suggestions. The founder has no preference on which one.
> - **Recipe sources:** a starter recipe library, plus saving recipes from any website by pasting a link. Whether importing recipes from other sites is legal is an open question.
> - **Online grocery ordering (e.g. Instacart, Tesco):** desirable later, not v1.
> - **Family calendar:** a nice-to-have for showing dinner on the family calendar, not v1.
> - Otherwise the product stands alone (confirmed)."
> -- BRIEF.md, ## Ecosystem & Integrations

### From assumptions-constraints.md -- Non-Functional Expectations

> ASMP-22: "Responsiveness: the shared grocery list feels instant when items are ticked or added, and a meal swap completes and reflects in the list within seconds." Basis: BRIEF.md's Scale & Non-Functional Expectations section states the list "must feel instant," and the Vision states swaps update the list "instantly."
> ASMP-23: "Responsiveness for heavier moments: a weekly plan generates within well under a minute with an explained wait, and recipe search results appear within about a second." Basis: the States fields of AI Weekly Dinner Plan Generation (FEAT-03) and Recipe Library (FEAT-08), and the brief's under-10-minutes-a-week planning goal (BRIEF.md, Success Criteria).
> ASMP-24: "Data volume and growth: several thousand households in the first year, 2-6 members each, with the product staying equally responsive as that base grows; every household's history is kept for the life of the account." Basis: BRIEF.md's Scale & Non-Functional Expectations, Order of magnitude; history depth per scope-boundaries.md SC-18.
> ASMP-25: "Resilience: the shared grocery list keeps working, with changes queued and synced later, when used in a supermarket with poor or no signal; an already-generated plan stays viewable offline." Basis: BRIEF.md's Scale & Non-Functional Expectations, Performance/availability.
> ASMP-26: "Privacy posture: children's data is minimal, parent-controlled, and used only for the household's own plan; no data is ever sold or used for advertising to any household member. A kid profile holds only a first name or nickname, an age band, and dietary rules; the organiser can see every time support viewed the household." Basis: BRIEF.md's Privacy section under Scale & Non-Functional Expectations and Constraints.
> ASMP-27: "Compliance: because the product processes children's data in the US and UK, it is treated as subject to children's-privacy-class protections (minimal collection, verifiable parental consent when a kid profile is created, parental control, no behavioral advertising to minors) alongside general personal-data protection for all household members, including the right to a copy of their data and to deletion; no medical-data regime applies, since the product gives no medical or diet advice." Basis: domain reasoning from decomposition-checklists.md's Compliance section, grounded in BRIEF.md's stated children's-privacy sensitivity and its explicit "no medical or diet advice" positioning.
> ASMP-28: "Localization: measurement units, currency, and supermarket aisle names are configurable per household to support both US and UK households from launch; the product is in English only." Basis: BRIEF.md's Scale & Non-Functional Expectations, Geography; English-only per scope-boundaries.md SC-13.
> ASMP-29: "Accessibility and one-handed use: every primary action is reachable with one thumb on a phone, tap targets are large, text stays readable with the phone's larger-text settings, and safety badges and ineligibility reasons are conveyed in words, not by color alone." Basis: BRIEF.md's Constraints, Design preference ("big tap targets for one-handed use in a shop").

### From assumptions-constraints.md -- Dependencies

> ASMP-30: "AI text/plan-generation capability" -- Required to generate the weekly dinner plan and swap alternatives on the paid tier; without it, planning would revert to a fully manual, unassisted experience, and the product's paid promise would not exist.
> ASMP-31: "Device-notification delivery capability" -- Required for the weekly "plan ready" notification, the daily "tonight's dinner" nudge, and swap-suggestion alerts; without it, these degrade to email (plan ready) or in-app-only discovery, weakening the product's proactive rhythm.
> ASMP-32: "Transactional email capability" -- Required for account sign-up and sign-in recovery, the plan-ready email fallback, billing and grace-period notices, data-export and deletion confirmations, safety-concern reports reaching the operator, and support acknowledgements; without it, several account, billing, and safety messages would have no reliable route.
> ASMP-33: "Payment-processing capability" -- Required to run the paid household subscription (monthly or yearly); without it, the product could offer only the free tier and would have no path to the revenue the founder needs within about three months.
> ASMP-34: "Recipe/food-content data capability" -- Required to seed the starter recipe library at launch with complete ingredient data the allergy check can verify; without it, a brand-new household would have to rely entirely on manually imported recipes before any plan could be generated.
> ASMP-35: "Real-time data-synchronization capability" -- Required for the shared grocery list and plan to update live across household members' devices and to reconcile changes made while offline; without it, the list would revert to the unreliable, manually-reconciled experience the brief describes households currently suffering through.
> ASMP-36: "Web-page recipe extraction capability (v1)" -- Required for Recipe Import from Web Link (FEAT-10) to read a recipe's ingredients and steps from a pasted link; without it, households could still add recipes by manual entry. Whether importing from other sites is legally acceptable, and in what form, remains a brief open question to settle before v1 (BRIEF.md, Open Questions).
> ASMP-37: "Online grocery-ordering and family-calendar capabilities (Later)" -- Required only for Online Grocery Ordering Handoff (FEAT-20) and Family Calendar Sync (FEAT-21); the core product works fully without them.

### From feature-dependency-map.md -- External Touchpoints

> "Every row traces to the `## Dependencies` section of `assumptions-constraints.md` (ASMP-30 to ASMP-37). ASMP-37 names two separate Later-phase capabilities and is therefore split into two rows. Integration Specs were back-filled per analysis batch after Brief validation. The final analysis batch (FEAT-22) ran the full coverage check across all 25 validated Briefs: every row has at least one covering Integration spec, and all 15 Integration specs inventoried across the Briefs map to a row here (no new capability category was discovered). FEAT-22 inventories no Integration spec and has no External Touchpoints row of its own. Its support-visit note (FEAT-22.SPEC-009) is in-app. The safety-concern emails, to the operator and for the household's resolution notice, are specified in FEAT-02.SPEC-010."

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

## 2. Non-Functional Requirements & Scale Design

The demand side of this architecture, sourced from profile Section 7 (Demand-Side Inputs). Values not carried upstream are marked "Assumption —".

| Dimension | Expectation | Source |
|-----------|-------------|--------|
| Launch load | "Several thousand households in the first year, with 2-6 people each" — roughly 5,000-25,000 member profiles by end of year one; mobile-first use in short bursts ("People plan on the sofa and shop with the phone in one hand") | Profile Section 7, BRIEF.md Scale & Non-Functional Expectations; ASMP-24 |
| Growth trajectory | "The product staying equally responsive as that base grows; every household's history is kept for the life of the account" — data grows by about 52 archived weeks per household per year (FEAT-19, FEAT-25 feasibility notes). Assumption — no upstream figure beyond year one; the design targets headroom to the low tens of thousands of households without re-platforming, since nothing upstream predicts faster growth | Profile Section 7, ASMP-24; Assumption — year-two figure not stated upstream |
| Performance targets | Grocery list "feels instant when items are ticked or added"; propagation to another device within 2 seconds (FEAT-06 feasibility); "a meal swap completes and reflects in the list within seconds" (tap-to-updated under 10 seconds per FEAT-04); "a weekly plan generates within well under a minute with an explained wait"; "recipe search results appear within about a second" | Profile Section 7, ASMP-22, ASMP-23; technical-feasibility.md FEAT-04, FEAT-06 |
| Availability posture | "No specific uptime number was stated." The shared grocery list "must keep working in a supermarket with bad signal. Changes made offline sync later"; "an already-generated plan stays viewable offline." Assumption — single-region managed hosting with provider-standard uptime is adequate, because the offline-first client (ASMP-25) masks short server outages for the two most-used surfaces | Profile Section 7, BRIEF.md Performance / availability; ASMP-25; Assumption — no uptime figure upstream |
| Security posture | "Children's data is minimal, parent-controlled and never used for anything but the family's own plan. There are no ads and no data selling." A kid profile holds only a first name or nickname, an age band and dietary rules; "the organiser can see every time support viewed the household." Dietary Rule (allergy) data is the most sensitive data class; payment details stay with the processor | Profile Section 7, BRIEF.md Privacy; ASMP-26 |
| Compliance obligations | Children's-privacy-class protections in the US and UK ("minimal collection, verifiable parental consent when a kid profile is created, parental control, no behavioral advertising to minors") plus general personal-data protection "including the right to a copy of their data and to deletion"; "no medical-data regime applies." Deletion completes within 30 days (FEAT-18.SPEC-008) | Profile Section 7, ASMP-27; technical-feasibility.md FEAT-18 |
| Cost envelope | "The AI cost per household must stay small, roughly one weekly plan plus a few swaps. The free tier gets no AI." Fixed platform cost stays inside the founder's sub-$100/month pre-revenue budget (SC-16, cited in technical-feasibility.md FEAT-08 and Open Question 5) | Profile Section 7, BRIEF.md Cost; technical-feasibility.md |
| Localization | "Measurement units, currency, and supermarket aisle names are configurable per household to support both US and UK households from launch; the product is in English only" | Profile Section 7, ASMP-28 |
| Accessibility | "Every primary action is reachable with one thumb on a phone, tap targets are large, text stays readable with the phone's larger-text settings, and safety badges and ineligibility reasons are conveyed in words, not by color alone" | Profile Section 7, ASMP-29 |

The binding drivers are four. First, **offline-tolerant real-time collaboration** on the shared grocery list and plan (ASMP-22, ASMP-25, ASMP-35; feasibility verdict Hard for FEAT-06). It shapes the client architecture (service worker, IndexedDB outbox, server-cache library), the realtime transport, and the decision to order concurrent edits with server-assigned sequence numbers instead of client clocks. Second, **children's-privacy-class data handling** (ASMP-26, ASMP-27). It favours enforcing household isolation in the database itself (row-level security), minimising the number of third-party processors that hold member data, scrubbing telemetry, and choosing vendors with zero- or short-retention terms. Third, **the cost envelope** (BRIEF.md Cost, SC-16). It favours consolidating Database, Auth, Realtime and Storage under one managed Postgres platform, and it keeps AI spend proportional to paid households only. Fourth, **the fail-closed safety engine inside tight latency budgets** (feasibility verdict Hard for FEAT-02; ASMP-22, ASMP-23). It puts per-recipe allergen sets precomputed at ingest, plus deterministic pre-filtering before any LLM call, into the data and AI designs.

Raw load is not a binding driver. Several thousand households at 2-6 members is comfortably inside the free or entry tiers of every managed candidate in the landscape. The largest volume, the daily 4:00 pm nudge (FEAT-13.SPEC-001), is a few thousand messages a day. Scheduled bursts at default arrival slots (Sunday evening) are the one load shape that needs explicit handling: staggered dispatch and per-provider concurrency limits in the job layer.

## 3. Technology Stack Decisions

The 7 core stack decision areas, each in the ADR shape. Options come from technology-landscape.md Section 2; each selection was checked against the landscape's Section 3 Cross-Area Compatibility Notes and the decision guide's compatibility sanity rules.

### Frontend Framework

| Field | Value |
|-------|-------|
| Context | Scale: Medium (25 features, 196 specs); Interaction Complexity: Large (46 Automation specs, real-time and collaboration signals present); 60 Screen specs; Offline signal: Yes (FEAT-01.SPEC-013, FEAT-06.SPEC-008). BRIEF.md: "a mobile-first responsive web app... There are no native apps in v1." Section 2 drivers: instant list interactions (ASMP-22), about 1-second search (ASMP-23), offline resilience (ASMP-25), one-thumb accessibility (ASMP-29). Feasibility: FEAT-06 Hard (live sync + offline reconciliation); FEAT-07/FEAT-13 depend on Web Push, which on iOS requires a home-screen-installed web app |
| Recommended | Next.js (React), App Router, current stable major version, delivered as an installable Progressive Web App (web app manifest + service worker) |
| Rationale | The landscape entry notes Next.js has the "largest ecosystem for real-time client updates and PWA/offline tooling". Both are needed: the Real-time and Offline signals are Yes, and FEAT-06 is rated Hard. Installability is also the only route to iOS Web Push for the FEAT-07/FEAT-13 notification rhythm (feasibility risk "Web Push reach on iOS"). Server Components keep the 60-screen client bundle small for low-signal mobile use (ASMP-22/23). Route Handlers and Server Actions serve the Medium-scale API surface without a second deployable (see Backend / API Layer). React binds the strongest component-layer and state-management options (shadcn/ui, Radix, TanStack Query), per landscape Section 3 |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| SvelteKit | Open source, free; hosting cost identical to Next.js | Low -- Vite-based, adapters for every host researched | High -- no practical ceiling at several thousand households | Low-Medium -- Svelte component model does not port to React | Specialist -- Svelte skills (6.9% professional adoption per landscape) | Field measurements on target low-end phones show the Next.js client bundle missing the ~1-second search or instant-list targets (ASMP-22/23), and the team already knows Svelte |
| Remix (React Router v7) | Open source, free | Low-Medium -- a server runtime to host; fewer zero-config hosts than Next.js | High | Low -- web-standard loaders/actions, React components portable | Mainstream React | The team wants web-standards-first progressive enhancement and a lighter framework layer than Next.js, and accepts building PWA/offline tooling with less prebuilt ecosystem support |
| Nuxt 3 (Vue) | Open source, free | Low -- mature module ecosystem | High | Medium -- Vue components and Pinia do not port to React | Mainstream Vue (smaller talent pool than React per landscape) | The implementing team is Vue-first; then pair it with Pinia, Headless UI and Vite per the landscape's binding notes |

### Backend / API Layer

| Field | Value |
|-------|-------|
| Context | Frontend: Next.js (ADR-001). 15 Integration specs and 46 Automation specs; 20 cross-feature business rules (XBR). Background processing signal: Yes, with 7 scheduled automations (FEAT-03.SPEC-003, FEAT-07.SPEC-001, FEAT-13.SPEC-001, FEAT-06.SPEC-004, FEAT-25.SPEC-003, FEAT-09.SPEC-006, FEAT-04.SPEC-005). Inbound webhooks from payment processing (FEAT-14.SPEC-009 Inbound Events). Section 2 cost envelope (sub-$100/month pre-revenue). Feasibility: FEAT-02's safety engine "is application logic in whichever Backend / API Layer option is selected" |
| Recommended | Next.js Route Handlers and Server Actions on the Node.js runtime as the single TypeScript backend, with a framework-independent domain layer (`src/features/*/rules`, `src/features/*/actions`, `src/server/*`). All long-running and scheduled work goes to the durable job platform (ADR-013), not to request handlers |
| Rationale | The landscape entry says co-location "avoids a second deployable for a medium-scale API surface" (Scale: Medium). It flags the risk that effort rises "as Integration-spec count (15) and background-job count grow". That risk is handled structurally: scheduled and multi-step automations run as Inngest functions (ADR-013) served from the same codebase, and domain logic sits in plain TypeScript modules, so it could move to a NestJS/Fastify service without rewrites. One language end to end shares types and validation schemas between client and server across 17 entities and 54 relationships, which also keeps the offline outbox and server merge rules in sync (FEAT-06.SPEC-008) |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| NestJS (on Node.js), separate service | Open source; adds a second always-on service (~$25/month Render starter per landscape) | Medium -- second deployable, CORS/auth propagation between services | Very high -- scales independently of the web tier | Low -- runs on Express or Fastify | Mainstream TypeScript plus Nest's module/DI conventions | The team grows past ~5 backend engineers, or a native mobile app joins the web app as a second API client, so a stable versioned API service pays for itself |
| Fastify, separate service | Open source; second service cost as above | Medium -- second deployable, less structure imposed | Very high -- 2-3x Express throughput per landscape | Low | Mainstream Node | Measured API latency on the grocery-list hot path misses ASMP-22 on serverless functions and a persistent low-latency Node process is needed |
| Django (Python) | Open source; separate Python service | Medium-High -- two languages and runtimes, no shared types | Very high | Low framework lock-in; high cost to switch languages later | Python plus a separate TypeScript frontend skill set | The team is Python-first and wants Django's admin for the FEAT-08.SPEC-004 content pipeline and FEAT-22 operator tooling more than it values shared TypeScript types |

### Database

| Field | Value |
|-------|-------|
| Context | Data Complexity: Medium -- 17 entities, 54 inter-entity relationships, 20 XBR rules; household-scoped multi-tenancy with role-based access (Authentication signal: Yes, role-based); contention rules on 16 of 17 entities; Compliance/privacy signal: Yes (children's-privacy-class data, ASMP-26/27). Section 2: several thousand households (ASMP-24), history kept for life, sub-$100/month pre-revenue budget. Real-time signal: Yes, with live sync (ASMP-35). Landscape Section 3 note: selecting Supabase "consolidates Auth, Storage, and Realtime under one Pro-tier bill ($25/month base)" |
| Recommended | Supabase (managed PostgreSQL), Pro plan, single region (US East, to serve US and UK users from one region; see ADR-032) |
| Rationale | A relational engine fits the dense 17-entity, 54-relationship graph and the transactional conditional writes behind the reject-with-refresh and per-slot lock rules (FEAT-04.SPEC-009, FEAT-23.SPEC-006). Supabase adds Postgres row-level security, which enforces household isolation and kid-data restrictions in the database itself: the strongest control for the Compliance/privacy signal and for FEAT-22's "read-only must be enforced server-side" risk. The same platform supplies Realtime (ADR-015), Auth (ADR-029) and Storage (ADR-008) at one $25/month base, the only combination in the landscape that fits the sub-$100/month budget while covering four active areas. Standard Postgres wire compatibility keeps the data portable to Neon or RDS (landscape Section 3) |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Neon (Serverless Postgres) | Usage-based, $0.106/CU-hour (Launch) + $0.35/GB-month; scale-to-zero | Low -- managed, branching per preview environment | High -- serverless autoscaling | Low -- plain Postgres | Mainstream Postgres | The FEAT-06 spike selects Ably (or another non-Supabase transport) and a non-Supabase identity provider, so no Supabase bundle remains to justify it; Neon's per-branch preview databases then give the better dev workflow |
| Amazon RDS for PostgreSQL | $30-140+/month with Multi-AZ and storage | Medium -- VPC, IAM, parameter groups | Very high | Low data lock-in; moderate operational lock-in to AWS | Mainstream Postgres plus AWS operations | A contractual or regulatory requirement demands Multi-AZ failover, private networking, or a specific data-residency region that Supabase cannot meet |
| PlanetScale for Postgres | From $5/month single node; 3-node HA from $15/month | Low | High | Low -- Postgres wire protocol | Mainstream Postgres | The team wants the cheapest HA Postgres and has already chosen standalone auth, realtime and storage vendors |

### ORM / Data Access

| Field | Value |
|-------|-------|
| Context | Database: Supabase Postgres (ADR-003) with row-level security; backend: Node/TypeScript (ADR-002); 17 entities, 54 relationships; schema evolution across 25 features; serverless function runtime (ADR-032), so connection pooling and a light client matter |
| Recommended | Drizzle ORM with Drizzle Kit (generated SQL migration files), connecting through Supabase's transaction-mode connection pooler |
| Rationale | The landscape entry lists a code-first TypeScript schema with inferred types, no generation step, a lighter runtime that is "serverless-friendly", and output as plain SQL. Plain SQL matters here because row-level-security policies, database functions (per-slot conditional writes, household sequence counters) and Realtime publication settings must live in the same versioned migrations as the tables. Drizzle Kit's weaker handling of complex schema changes (landscape) is offset by reviewing generated SQL and hand-editing it where needed. Drizzle pairs with the Node backend and Postgres per landscape Section 3 |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Prisma ORM (v7) | Open source core; Accelerate/Prisma Postgres priced separately | Low -- Prisma Migrate and Studio | High -- Prisma 7 removed the Rust engine overhead | Moderate -- `.prisma` DSL and generated client | Mainstream | The team values Prisma Migrate's schema-diff workflow and Prisma Studio over SQL-first control, and is willing to maintain RLS policies in raw SQL migrations alongside it |
| TypeORM | Open source, free | Medium -- decorator-heavy entities | High | Low-Moderate | Mainstream, older patterns | The backend moves to a NestJS service (ADR-002 alternative) and the team prefers NestJS's first-party TypeORM integration |

### CSS / Styling

| Field | Value |
|-------|-------|
| Context | 60 Screen specs; mobile-first, one-handed UI with large tap targets, larger-text support, and safety badges "conveyed in words, not by color alone" (ASMP-29); Interaction Complexity: Large; no user-supplied design system (`design_system_source: none`, see Section 9); frontend: Next.js with Server Components (ADR-001) |
| Recommended | Tailwind CSS v4 |
| Rationale | The landscape entry lists zero runtime cost, "full Server Component support", pairing with all frameworks researched, and the lowest onboarding barrier. That fits a 60-screen build where most screens are built by composing a shared component layer (Section 9). Utility classes make the ASMP-29 constraints (minimum 44px tap targets, rem-based type that scales with OS text settings) enforceable as a small set of shared tokens in the Tailwind theme. Tailwind is also the prerequisite of the chosen component layer (shadcn/ui, ADR-027) |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| CSS Modules (vanilla CSS) | Free, native | Low | High | Zero | Mainstream CSS | The downstream builder brings its own design system authored as plain CSS custom properties and wants no utility-class layer |
| vanilla-extract | Open source, free | Medium -- TypeScript-authored styles | High | Low -- outputs static CSS | TypeScript-fluent frontend engineers | A future user-supplied design system arrives with a large typed token set that benefits from compile-time token checking |
| Panda CSS | Open source, free | Medium -- newer tool | High | Low | Moderate | The team prefers style-props authoring compiled to static utilities, and accepts a smaller ecosystem than Tailwind |

### State Management

| Field | Value |
|-------|-------|
| Context | Real-time signal: Yes (FEAT-03.SPEC-011, FEAT-06.SPEC-005, FEAT-01.SPEC-004); Collaboration/concurrency: Yes (16 of 17 entities); Offline: Yes (FEAT-01.SPEC-013 drafts, FEAT-06.SPEC-008 queued edits); Interaction Complexity: Large; Complex forms: No. Feasibility: offline local persistence and queued sync across FEAT-01, 03, 05, 06, 09, 11, 12 and 17 (cross-feature theme) |
| Recommended | TanStack Query (server state, with IndexedDB-persisted cache and paused offline mutations) + Zustand (client/UI state, including the persisted offline outbox and setup-wizard drafts) |
| Rationale | The landscape calls this "the documented 2026 default pairing". TanStack Query owns the plan, list and recipe caches, which realtime events patch or invalidate. Its persisted cache and paused-mutation model are the building blocks of offline plan viewing and queued list edits (ASMP-25). Zustand holds UI state and a durable outbox of idempotent operations for replay on reconnect, which handles FEAT-01.SPEC-013's rule that replayed saves must surface a retry state and never be dropped silently. Both are React-compatible (landscape Section 3) |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Redux Toolkit + RTK Query | Open source, free | Medium -- more boilerplate | High | Moderate -- server cache coupled to the Redux store | Mainstream React/Redux | The team wants one store and DevTools timeline for both server and client state, for example to debug the offline replay order across devices |
| Jotai (with TanStack Query for server state) | Open source, free | Low-Medium | High | Low | Atom mental model | Fine-grained per-field draft state in setup screens becomes the dominant client-state need and store-based drafts cause re-render issues |

### Build Tooling

| Field | Value |
|-------|-------|
| Context | Frontend: Next.js (ADR-001); 25-feature, 196-spec codebase with frequent iteration; decision guide: "meta-frameworks ship their own build pipeline; overriding it is a decision that needs a documented driver" |
| Recommended | Next.js bundled pipeline: Turbopack for development, and the framework's default production build. No custom bundler configuration |
| Rationale | The landscape entry for Turbopack says it is "purpose-built for Next.js... 9.5x faster incremental builds than Webpack in dev". No profile metric justifies overriding the bundled pipeline, and landscape Section 3 notes Turbopack is only meaningful with Next.js, which ADR-001 selects |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Vite 8 (with Rolldown) | Open source, free | Low | High | Low | Mainstream | The frontend switches to SvelteKit, Nuxt or Remix (ADR-001 alternatives), where Vite is the native pipeline |
| Webpack 5 (Next.js legacy mode) | Open source, free | High -- configuration overhead | High | Low | Webpack plugin expertise | A required build plugin (for example a service-worker generator) is only available for Webpack and has no Turbopack-compatible equivalent |
| Rspack | Open source, free | Medium -- newer plugin ecosystem | High | Low | Moderate | The project leaves Next.js for a custom React setup and needs a Webpack-compatible, faster bundler |

## 4. Platform & Service Decisions

### File & Object Storage

| Field | Value |
|-------|-------|
| Context | Import/export signal: Yes -- FEAT-18.SPEC-001 Export Household Data and FEAT-18.SPEC-006 Export Generation Processing (generated JSON/CSV-class export files); File upload signal: No ("No spec references an image, photo, or media upload"). FEAT-10 import stores extracted text, not files. Feasibility FEAT-18: the export file "concentrates the most sensitive data, including kids' allergy data", so download links need expiry and access control; deletion within 30 days must cover "previously generated export files" |
| Recommended | Supabase Storage (included in Supabase Pro, ADR-003): a private bucket per environment, household-scoped object paths, short-lived signed download URLs, and a scheduled purge job (ADR-013) that deletes export files after a fixed retention window |
| Rationale | Only one low-volume use case exists (occasional household exports), so a separate storage vendor adds a processor inside the children's-data boundary without adding capability. Supabase Storage is bundled at no extra base cost (landscape Section 3) and its access policies reuse the same RLS household model. Signed URLs with expiry meet the feasibility access-control requirement |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Cloudflare R2 | $0.015/GB-month, $0 egress | Low -- S3-compatible API | Very high | Low -- S3 API | Mainstream | The product adds user media (recipe photos) with heavy download traffic, where zero egress dominates cost |
| Amazon S3 | $0.023/GB-month, ~$0.09/GB egress | Low | Very high | Low -- de facto standard | Mainstream | The team standardises on AWS and wants S3 lifecycle policies to auto-expire export files without a purge job |
| Backblaze B2 | Below S3 pricing | Low -- S3-compatible | High | Low | Mainstream | Cost-driven migration away from Supabase Storage once stored volume grows past the Pro plan's included 100GB |

### Email & Messaging Delivery

| Field | Value |
|-------|-------|
| Context | Notifications signal: Yes -- 20 Notification specs; channels are push, email and in-app. ASMP-31 device-notification delivery (plan-ready FEAT-07.SPEC-002, tonight's nudge FEAT-13.SPEC-002/004, swap alerts FEAT-04.SPEC-006); ASMP-32 transactional email (FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-14.SPEC-012, FEAT-18.SPEC-012). Target: 80% of plan-ready messages delivered within one minute (FEAT-07 feasibility). Section 2: children's-privacy-class handling; kids and the operator are never push recipients (XBR-13). The in-app inbox is a product surface stored in the primary database |
| Recommended | Two providers behind one internal notification module: **Resend** for transactional email (free tier 3,000/month, then paid tier) and **OneSignal** for Web Push (free tier). Channel fallback (push to email, FEAT-07.SPEC-006) and deduplication (at most once per member per household-week, FEAT-07.SPEC-002) stay in application code |
| Rationale | Resend's landscape entry notes a "developer-friendly API and template tooling for transactional email" with a permanent free tier. That covers launch volume (account, recovery, billing, export and safety emails) at $0 inside the SC-16 budget, with an official Node SDK (landscape Section 3). OneSignal is the "quickest self-serve push setup" for Web Push to a web-only product (no native apps, SC-05). Keeping fallback and dedupe in-app, rather than in an aggregator, keeps the explicitly specified delivery rules testable in one place and avoids Courier's added abstraction layer |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Postmark (email) | Free 100 emails/day; from $19.95/month for 50,000 emails | Low | High | Low-Moderate -- message-stream model | Mainstream | Deliverability of time-sensitive recovery and grace-period emails measurably underperforms (spam placement, delay), since Postmark is transactional-only with sub-2-second delivery |
| Courier (unified push + email) | $0.005/message, 10,000/month free | Low-Medium -- extra abstraction layer | High | Moderate -- routing/template model | Mainstream | Notification rules multiply (more channels, per-member preferences, SMS) and the team would rather configure routing and fallback centrally than maintain it in code |
| SendGrid (Twilio) | No free plan; usage tiers | Low-Medium | Very high | Moderate -- coupled to a Twilio account | Mainstream | An SMS channel is added to the product, making Twilio's bundled email + SMS worth a single vendor |

### Payments & Billing

| Field | Value |
|-------|-------|
| Context | Payments/billing signal: Yes -- FEAT-14.SPEC-001 through FEAT-14.SPEC-012 (tiers, upgrade monthly/yearly, downgrade/cancel at period end, 7-day grace period, refunds, inbound renewal/retry events); FEAT-01.SPEC-011 free-tier default. ASMP-33: revenue needed "within about three months". Billing in USD and GBP (ASMP-28). Feasibility: webhooks are at-least-once and out of order; the 7-day grace window must be reconciled with processor retries; Open Question 9 (merchant of record vs processor) |
| Recommended | Stripe Billing: Stripe-hosted Checkout for upgrade, Stripe Customer Portal for payment-method management, Stripe Subscriptions with USD and GBP prices, and signed webhooks ingested idempotently (ADR-026). The 7-day grace period (FEAT-14.SPEC-007) is enforced in Plateful's own billing state machine, with Stripe's Smart Retries configured to fit inside it |
| Rationale | The landscape rates Stripe "very mature, the default payments integration for the frameworks researched" with the lowest per-transaction cost among the options (2.9% + $0.30, plus 0.7% Billing). That matters for a low-price household subscription, where Paddle's $0.50 fixed fee takes a much larger share of each charge. Hosted Checkout and Portal cover most of FEAT-14.SPEC-002/003's UI and keep card data out of Plateful, which fits the three-month revenue timeline. FEAT-14's grace, refund and tier-gating rules are product-specific enough to need code under any option. Sales-tax/VAT ownership remains an open founder decision (feasibility Open Question 9); the Paddle row below states when to switch |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Paddle (merchant of record) | 5% + $0.50 per transaction, all-inclusive | Low -- tax, VAT and chargebacks handled | High | High -- Paddle is seller of record | Mainstream | The founder decides not to own US sales-tax and UK VAT registration and filing (feasibility Open Question 9); in that case the per-transaction premium buys the compliance surface away |
| Chargebee (on top of Stripe) | Free to $250,000 cumulative billing; $599/month Performance plan | Medium -- second billing platform | High | Moderate -- billing data model | Mainstream | Pricing experiments (coupons, trials, multiple paid tiers) outgrow the hand-built FEAT-14 state machine and billing logic changes weekly |
| Recurly | Revenue-based tiers | Medium | High | Moderate | Mainstream | Involuntary churn from failed renewals becomes material and specialised dunning beats Stripe Smart Retries inside the 7-day grace window |

### AI & Intelligent Behavior

| Field | Value |
|-------|-------|
| Context | AI/ML behavior signal: Yes -- FEAT-03.SPEC-010 plan generation, FEAT-04.SPEC-007 swap alternatives, FEAT-12.SPEC-004 preference weighting (paid tier). BRIEF.md: "The founder has no preference on which one"; "the AI cost per household must stay small, roughly one weekly plan plus a few swaps. The free tier gets no AI." ASMP-23: generation "well under a minute with an explained wait". Feasibility: FEAT-03 **Research-spike recommended** (full-week success rate, p95 latency, tokens per run, Sunday-evening burst behaviour); FEAT-04 alternatives within "a couple of seconds"; dietary constraints leave the product, so provider retention terms fall under children's-privacy-class handling |
| Recommended | Anthropic Claude API: a Sonnet-tier model for weekly plan generation, using structured (JSON-schema) output and prompt caching of the static instruction and recipe-pool prefix. It sits behind a provider-agnostic `PlanGenerator` interface. Only the deterministically pre-filtered, FEAT-02-verified candidate pool is sent, never member names or ages. Swap alternatives (FEAT-04.SPEC-007) are served first from a deterministic ranking over the verified pool, with the LLM called only for re-ranking and scarcity explanation, within a strict timeout |
| Rationale | The landscape entry notes "strong structured-output and instruction-following for plan/swap generation" and prompt caching that "drops cache-hit cost to 0.1x base input". Every paid household's scheduled run shares the same instruction prefix, so caching directly serves the per-household cost envelope. Deterministic pre-filtering (safety, schedule, budget) before the call is the feasibility document's own mitigation: it shrinks tokens and failure modes, and FEAT-02 re-checks every returned candidate (out-of-pool results are discarded per FEAT-03.SPEC-010). The adapter plus the FEAT-03 spike keep the choice reversible. The spike should compare against the Gemini and OpenAI rows below before launch |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| OpenAI GPT API (mid-tier) | ~$2/$10 per 1M tokens; Batch API halves rates | Low | Very high | Low-Moderate -- REST API | Mainstream | The FEAT-03 spike shows the scheduled weekly run can move to asynchronous Batch processing ahead of arrival time, and the 50% batch discount beats prompt-cache savings |
| Google Gemini API (Flash tier) | Gemini 3.8 Flash $0.75/$3.75 per 1M tokens (promotional through 2026-12-31); 3.1 Pro $2/$12 | Low | Very high | Low-Moderate | Mainstream | The spike shows a Flash-tier model meets the full-week success rate, making it the cheapest option, or the swap-alternatives explanation call needs lower latency than the recommended tier delivers |
| Self-hosted open-weight model | GPU compute rather than per-token pricing | High -- hosting, scaling, model updates | Team-bound | Low data lock-in, high operational ownership | Specialist ML ops | Paid households reach a volume where GPU cost undercuts API spend, or a legal review requires dietary data never to leave Plateful's infrastructure |

### Search

| Field | Value |
|-------|-------|
| Context | Search signal: Yes, complexity "simple filter" -- FEAT-08.SPEC-001 ("Searching by recipe name or ingredient, and filtering by dietary badge"); "No spec references full-text search or faceted search." ASMP-23: results "within about a second". Feasibility FEAT-23 risk: the picker needs per-recipe safety verdicts for the household on every listing |
| Recommended | PostgreSQL full-text search on the primary database: a `tsvector` over recipe name and ingredient names with a GIN index, combined with filters on precomputed per-recipe allergen/diet-attribute arrays so the household's eligibility is computed in the same query |
| Rationale | The landscape describes native Postgres FTS as "free and sufficient for small-to-medium recipe catalogs and simple name/ingredient/badge filtering". The corpus is one starter library plus per-household imports (at most 30 per week, FEAT-10.SPEC-007). Keeping search in Postgres lets one indexed query join the safety-attribute arrays that address FEAT-23's one-second risk. An external index would need its own eligibility sync |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Typesense (Cloud) | From ~$30/month; self-host free | Medium -- index sync pipeline | High | Low -- open source | Moderate | Usage data shows failed searches from typos or near-miss ingredient names and typo tolerance becomes a product requirement |
| Meilisearch | Self-host free; Cloud from low hundreds $/month | Medium -- index sync | High | Low | Moderate | Same trigger as Typesense, plus a preference for Meilisearch's relevance defaults |
| Algolia | Free to 10,000 searches/month; Grow from $550/month | Low -- managed | Very high | Moderate | Low | Search becomes a primary discovery surface (for example a public recipe catalogue) that needs instant-search UI and merchandised ranking |

### Background Jobs & Scheduling

| Field | Value |
|-------|-------|
| Context | Background processing signal: Yes -- FEAT-03.SPEC-003 weekly generation, FEAT-07.SPEC-001 plan-ready trigger, FEAT-13.SPEC-001 daily 4:00 pm local nudge, FEAT-06.SPEC-004 week rollover, FEAT-25.SPEC-003 check-in cycle, FEAT-09.SPEC-006 14-day invitation expiry, FEAT-04.SPEC-005 suggestion lapse; Import/export: FEAT-18.SPEC-006 export generation; Notifications: 20 specs with queued-retry email. Feasibility: one generation in flight per household-week (FEAT-03.SPEC-010); Sunday-evening bursts hit provider rate limits; daily volume can exceed the 25,000-run free tier; daylight-saving transitions must not double-fire; 30-day deletion cascades (FEAT-18.SPEC-008) |
| Recommended | Inngest (managed durable workflows), served from the Next.js app's `/api/inngest` endpoint. Cron fan-out per timezone bucket (hourly cron → one batched event per due household), per-household concurrency keys, provider-level throttling for Claude/OneSignal/Resend, step-level retries, and idempotency keys for each notification send |
| Rationale | The landscape rates Inngest the "widely recommended default for early-stage Next.js/Node SaaS", with "step-based durability with retries" and "without a separate worker deployment". That fits a serverless-hosted Next.js backend (ADR-002, ADR-032) with 46 Automation specs, many of them multi-step (generation → leftover computation → list recalculation → plan-ready dispatch). Built-in concurrency keys enforce FEAT-03's one-run-per-household rule, and throttling spreads the Sunday-evening burst. Timezone-bucketed fan-out keeps run counts near the 25,000/month free tier at launch. The first paid tier ($75/month) arrives with paid-household revenue |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Trigger.dev | Free tier 50,000 runs/month | Low -- TypeScript tasks, no function timeouts | High | Lower -- open-source core, self-host option | Mainstream | Long-running generation or export steps exceed serverless function limits, or legal review requires job payloads (children's dietary data) to run on self-hosted infrastructure |
| BullMQ (self-hosted, Redis-backed) | Free library; Redis ~$5-10/month plus worker compute | Medium -- operate workers | High | Low | Mainstream Node | Hosting moves to always-on containers (ADR-032 Render/Fly.io alternatives) and flat cost beats per-run pricing at high daily nudge volume |
| Upstash QStash | 500 messages/day free, usage-based | Low -- HTTP delivery only | High | Low | Mainstream | Only simple timed HTTP triggers remain (no multi-step workflows), for example if generation moves to a separate service |

### Caching & Performance

| Field | Value |
|-------|-------|
| Context | Scale hints: Yes -- ASMP-22 (list "feels instant", swap "within seconds"), ASMP-23 (plan "well under a minute", search "about a second"), ASMP-24 (several thousand households); Offline: Yes -- FEAT-01.SPEC-013, FEAT-06.SPEC-008; ASMP-25 "keeps working... with changes queued and synced later". Feasibility: FEAT-02 latency risk (precompute allergen sets, cached verdicts risk stale passes); FEAT-06 iOS service-worker background-sync limits |
| Recommended | Client-side offline caching via a service worker and IndexedDB (PWA layer): app shell and current plan/list cached for offline use, a persisted TanStack Query cache, and an IndexedDB outbox of idempotent mutations replayed on reconnect and on app foreground (not relying on Background Sync, which iOS lacks). Server side: no separate cache service at launch; hot-path performance comes from Postgres precomputation (per-recipe allergen/attribute sets computed at ingest and edit, indexed) instead of a verdict cache |
| Rationale | The landscape's service-worker/IndexedDB entry "directly serves the offline-queuing requirement". Offline is the binding performance driver, and it can only be met on the device. On the server, the feasibility document warns that cached verdicts "create invalidation risk... a stale 'pass' could leak through". Precomputed, database-resident attribute sets give the latency benefit without a second copy that could go stale, preserving FEAT-02's fail-closed rule. Several thousand households do not need a Redis tier, and skipping it keeps cost inside SC-16 |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Upstash Redis (added server cache) | First 500,000 commands/month free, then $0.20/100,000 | Low -- HTTP client | High | Low | Mainstream | Measured p95 for the grocery-list or picker queries misses ASMP-22/23 after indexing, or rate-limit counters (FEAT-10.SPEC-007) and short-lived locks need sub-millisecond reads |
| Redis Cloud | From ~$5/month; ~$70/month production tiers | Low-Medium | Very high | Low | Mainstream | Command volume passes roughly 5-10M/month, where flat pricing beats Upstash (landscape) |
| Vercel KV | ~2x Upstash per-command cost | Low | High | Higher -- Vercel wrapper | Mainstream | A server cache is needed and the team prefers Vercel-integrated billing over a separate Upstash account |

### Real-time & Collaboration

| Field | Value |
|-------|-------|
| Context | Real-time signal: Yes -- FEAT-03.SPEC-011 live plan sync, FEAT-06.SPEC-005 live grocery list sync (tick visible on another device within 2 seconds), FEAT-01.SPEC-004 live member list; Collaboration/concurrency: Yes -- last-write-wins, reject-with-refresh and merge rules on 16 of 17 entities; FEAT-06 feasibility verdict **Hard**, with a spike recommended comparing "Ably vs Supabase Realtime"; Open Question 8: client event time vs server-assigned order. Household channels are 2-6 members (ASMP-24) |
| Recommended | Supabase Realtime (included in Supabase Pro), using private per-household Broadcast channels authorised by RLS. Every committed write to plan, list, pantry or membership bumps a per-household monotonically increasing `sync_seq` in the same transaction and broadcasts `{entity, id, sync_seq}` after commit. Clients apply events in `sync_seq` order and re-fetch on gaps. Offline replays resolve by server-assigned order, with client event time kept only as the FEAT-06.SPEC-008 tiebreak input recorded at server receipt |
| Rationale | The landscape notes Supabase Realtime needs "no separate realtime service" when Supabase is the database (ADR-003), inside the $25/month Pro bill. Ably's stronger delivery guarantees matter less once ordering comes from the database: a server-assigned sequence makes delivery idempotent and gap-detectable on any transport, and it answers the feasibility risk that "skewed device clocks can make an older edit win". The merge rules are custom application logic under any transport (feasibility FEAT-06). The FEAT-06 spike stays in the plan to confirm convergence on target mobile browsers and the documented concurrent-connection figure (~200 on Pro per landscape) against weekend shopping peaks |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Ably | ~$30/month for 200 concurrent connections | Low-Medium -- separate service | Very high | Moderate -- channel/presence model | Mainstream | The FEAT-06 spike shows Supabase Realtime dropping or delaying events under weekend-peak concurrency, or connection counts exceed the Pro plan's concurrency figure |
| Pusher (Channels) | 200,000 messages/day and 200 concurrent connections free | Low | High | Moderate | Mainstream | The team leaves Supabase for the database (ADR-003 alternative) and wants the cheapest simple household-channel transport |
| Socket.io (self-hosted) | Free library; hosting plus Redis for scaling | High -- connection scaling and reconnection owned by the team | High with Redis adapter | Zero vendor | Specialist realtime ops | Hosting moves to always-on containers and the team wants the sync/merge protocol and transport fully in-house |

### Analytics & Product Telemetry

| Field | Value |
|-------|-------|
| Context | Scale hints: Yes -- ASMP-24 "several thousand households in the first year... staying equally responsive as that base grows". Specs emit named signals (e.g. member_onboarding_started/completed, grocery_handoff_initiated/succeeded/failed per FEAT-15/FEAT-20 feasibility notes). Compliance/privacy: Yes -- "no data is ever sold or used for advertising" (ASMP-26); feasibility risk: telemetry and session replay can capture kid data |
| Recommended | PostHog Cloud (free up to 1M events/month): server-side and client-side events keyed by household ID and member ID only, with no names, ages or dietary values in properties. Session replay is disabled on household-data screens and entirely for kid and operator contexts. No advertising integrations |
| Rationale | The landscape entry notes PostHog "bundles analytics, session replay, and feature flags in one developer-controlled stack" with 1M free events and a self-host option. Feature flags are also needed to gate FEAT-10 (legal status open, ASMP-36) and the Later-phase FEAT-17/20/21. Several thousand households stay inside the free tier. The self-host path is the documented escape route if a privacy review requires household-linked events in-house (feasibility FEAT-15) |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Mixpanel | Free to 20M events/month | Low | High | Moderate -- event schema | Low | Event volume passes PostHog's 1M free events and the team only needs funnels and retention, not flags or replay |
| Amplitude | Free under 50,000 MTU; Growth from ~$49/month | Low-Medium | High | Moderate | Low | A dedicated growth analyst wants behavioural-cohort tooling for household engagement analysis |

### Geo & Maps

Not activated — profile Section 3: "No specs reference location or mapping behavior. The only word-boundary matches for "map" resolve to "dependency map" (a cross-reference to feature-dependency-map.md, unrelated to geographic mapping); no spec contains geolocation, GPS, proximity, or distance content."

### Internationalization

| Field | Value |
|-------|-------|
| Context | Internationalization signal: Yes -- FEAT-16.SPEC-001 units and currency (at least USD and GBP), FEAT-16.SPEC-002 aisle-name customisation, FEAT-16.SPEC-004 cross-feature value conversion; ASMP-28 "the product is in English only"; feasibility FEAT-16: fixed conversion factors at display time, currency as a display label with no exchange-rate conversion, used by plan cost, recipes, list, billing and check-in (XBR-11) |
| Recommended | next-intl for locale-aware number, date and currency formatting (en-US and en-GB locales from the household setting, English message catalogue only), plus an in-house shared `locale` module holding the fixed unit-conversion table and aisle-name mapping, imported by both client and server |
| Rationale | The landscape describes next-intl as "Server Component-native locale/number/currency formatting if Next.js is the frontend choice", with low setup for the two-locale scope. The core need is formatting and conversion, not translation (landscape: "a locale-data/formatting need more than a multi-language translation need"). The conversion table stays plain shared code so offline screens convert without a network call |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| FormatJS (react-intl) | Open source, free | Low-Medium | High | Low -- ICU standard | Mainstream | The product adds real translation with ICU plural/select messages and the team wants the most rigorous ICU implementation |
| react-i18next | Open source, free | Low-Medium | High | Low | Mainstream | The frontend moves off Next.js (ADR-001 alternatives) and a framework-agnostic library is needed |
| LinguiJS | Open source, free | Low-Medium -- macro tooling | High | Low | Moderate | Bundle size on low-end phones becomes the constraint and extracted-string workflows are wanted |

### Recipe & Food Content Data

| Field | Value |
|-------|-------|
| Context | Product-mandated -- ASMP-34: "Recipe/food-content data capability -- Required to seed the starter recipe library at launch with complete ingredient data the allergy check can verify"; FEAT-08.SPEC-004 Starter Recipe Content Seeding & Maintenance (batches, completeness gate, corrections, retirements). Feasibility: FEAT-02 **Hard**, with the engine "only as safe as its ingredient-to-allergen mapping"; Open Question 5, vendor licences for storing and redisplaying content versus "the sub-$100/month pre-revenue budget"; vendor tiers listed at $149-$999/month; Spoonacular and Edamam taxonomies flagged as lock-in |
| Recommended | Curated/licensed starter set + manual editorial seeding: an owned starter library of ingredient-complete recipes (name, quantity and unit on every ingredient per FEAT-02.SPEC-007), authored or licensed outright, and loaded through an internal batch-import pipeline (FEAT-08.SPEC-004) that runs the completeness gate and precomputes allergen attributes. It is paired with an in-house, versioned ingredient-to-allergen taxonomy with compound-term expansions, guarded by a labelled regression suite |
| Rationale | The landscape rates this option the "highest control over the allergen-completeness fail-closed policy (FEAT-02.SPEC-007)" with no vendor lock-in and no recurring fee. That matches the product's hardest risk (zero allergy incidents) and the sub-$100/month budget. Owning the content avoids the unresolved licensing question of storing and redisplaying vendor recipes (Open Question 5). The in-house taxonomy is needed anyway, because imported recipes (FEAT-10) carry free text no vendor taxonomy covers. The FEAT-02 spike measures whether a vendor taxonomy should later be layered underneath |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Edamam Recipe Search API | Nutrition analysis API from $299/month (up to $999/month) | Low-Medium -- seeding pipeline plus sync | High -- 2M+ indexed recipes | Moderate -- Edamam schema | Mainstream | The FEAT-02 spike shows Edamam's allergen analysis materially beats the in-house taxonomy's recall, and licence terms permit storing and redisplaying the content |
| Spoonacular Food API | From $149-$300/month by tier | Low-Medium | High -- ~365,000 recipes | Moderate -- ingredient taxonomy | Mainstream | Launch coverage targets need thousands of recipes faster than editorial production can deliver, and licence terms permit storage |
| Tasty/Yummly-class licensed partnership | Negotiated | Medium-High -- business development | High | Depends on licence | Business development | A media partner offers a bulk licence with ingredient-complete data on terms inside the budget |

### Web Page Recipe Extraction

| Field | Value |
|-------|-------|
| Context | Product-mandated -- ASMP-36: "Web-page recipe extraction capability (v1) -- Required for Recipe Import from Web Link (FEAT-10)... Whether importing from other sites is legally acceptable... remains a brief open question"; FEAT-10.SPEC-005 (failure path to manual entry, progress until timeout); extraction "in a few seconds"; 30 imports per household per week (FEAT-10.SPEC-007). Feasibility: recipe-scrapers is Python while the backend is Node; SSRF risk from fetching arbitrary URLs; the AI fallback sends page content to an LLM |
| Recommended | AI-fallback extraction (hybrid): a self-built TypeScript parser for schema.org/Recipe JSON-LD and Microdata runs first, and only on parse failure is the page's extracted main text sent to the AI & Intelligent Behavior provider (ADR-011) with a strict JSON schema. Runs server-side as an Inngest function with a hardened fetcher (allow only http/https, block private and link-local IP ranges after DNS resolution, size and time limits, no cookies). Shipped behind a PostHog feature flag, off until the legal question (ASMP-36) is resolved |
| Rationale | The landscape describes the hybrid as "structured data first, LLM fallback only for unstructured pages", keeping "per-import AI cost low", and notes JSON-LD alone is "reliable on 95%+ of recipe sites". A TypeScript parser stays in the single Node runtime (ADR-002), avoiding the separate Python worker that recipe-scrapers would need (feasibility FEAT-10). Reusing the Claude adapter keeps one AI vendor relationship. The feature flag separates the build from the legal decision without blocking v1: manual entry (FEAT-10.SPEC-003) works regardless |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| recipe-scrapers (open-source, Python) | Free; runs as a separate Python worker | Medium -- second runtime | High | Zero | Python | The FEAT-10 spike shows the self-built parser's success rate well below recipe-scrapers' site-specific coverage, making a small Python extraction worker worth running |
| Managed recipe-extraction API (Apify actors) | Usage-based per run/page | Low | High | Moderate -- marketplace actor | Low | Site-markup churn makes parser maintenance a recurring burden, and per-import cost stays inside the AI cost envelope |
| Custom JSON-LD/schema.org parser only (no AI fallback) | Engineering time only | Medium | High | Zero | Mainstream | The legal decision permits only structured-metadata extraction, or sending third-party page content to an LLM is ruled out |

### Online Grocery Ordering Integration

| Field | Value |
|-------|-------|
| Context | Product-mandated -- ASMP-37 (first capability), Later phase: "Required only for Online Grocery Ordering Handoff (FEAT-20)... the core product works fully without them"; BRIEF.md "Online grocery ordering (e.g. Instacart, Tesco): desirable later, not v1." FEAT-20.SPEC-002: list payload out; acceptance/rejection and item availability in. Feasibility FEAT-20: **Research-spike recommended** -- Instacart covers the US only and no self-service UK grocer API was found |
| Recommended | Instacart Developer Platform (IDP) for US households in the Later phase, behind a `GroceryOrderingProvider` interface. The handoff is offered only where a provider covers the household's market (UK households see no handoff option until a UK partner is contracted). The partner-discovery spike runs before the Later phase starts |
| Rationale | The landscape describes Instacart IDP as "explicitly named in BRIEF.md; self-service developer platform... matching FEAT-20.SPEC-002's payload/response shape, for the US market". It is the only self-serve option. Vendor selection happens here, and the spike only covers integration mechanics (list-line to catalogue-SKU match rate) and UK partner availability, as the feasibility document recommends. The provider interface lets a UK partner be added without touching FEAT-20's screens or rules |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Direct retailer partnership (e.g. Tesco) | Negotiated | High -- business development plus custom integration | Retailer-bound | Depends on contract | Integration engineering plus business development | UK households are a majority of paid users and a UK grocer offers API access on workable terms |
| Grocery aggregator APIs (marketplace-listed) | Usage-based per listed API | Medium -- evaluate each | Varies | Varies per provider | Mainstream | The partner spike finds an aggregator giving UK retailer coverage with acceptable item-match rates, avoiding a direct partnership |

### Family Calendar Integration

| Field | Value |
|-------|-------|
| Context | Product-mandated -- ASMP-37 (second capability), Later phase; BRIEF.md "Family calendar: a nice-to-have for showing dinner on the family calendar, not v1." FEAT-21.SPEC-002: connect/disconnect, per-night entry create/update, inbound sync-outcome and disconnect events; one calendar per household (FEAT-21.SPEC-004); feasibility FEAT-21 Standard-with-integration, with Google OAuth verification lead time and household-deletion disconnect (FEAT-18.SPEC-008 step 5) |
| Recommended | Google Calendar API (direct) for the Later phase, behind a `CalendarProvider` interface, with OAuth tokens encrypted at rest and revoked in the FEAT-18 deletion cascade. Start Google's OAuth verification for the calendar-write scope early, because of the review lead time |
| Rationale | The landscape notes Google Calendar API is "free and sufficient if the household's calendar is Google Workspace/Gmail-based, which is plausible for a consumer household product". FEAT-21's scope (one calendar, per-night entries) is narrow. At $0 it avoids Cronofy's $99/month minimum for a nice-to-have feature, while the provider interface keeps a unified API available when demand shows |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Cronofy | $99/month minimum; Starter from $139/month | Low -- one API across Google, Microsoft, Apple | High | Moderate -- unified event model | Low | Connect-rate telemetry or support requests show a material share of households on Apple iCloud or Microsoft calendars |
| Direct multi-provider (Google + Microsoft Graph) | Free APIs; engineering time | High -- two OAuth integrations | High | Zero third-party | Mainstream | Microsoft calendar demand appears, Apple demand does not, and the team prefers no unified-API vendor |
| Nylas | Per-connected-account pricing | Low-Medium | High | Moderate-High (forced-migration signal per landscape) | Low | The product later adds email or contact integrations that justify Nylas's broader bundled scope |

## 5. Project Structure

Suggested starting structure for the recommended stack (Next.js App Router, ADR-001), adapted from the reference Next.js tree. It is a single package (ADR-022). URLs are product-oriented (`/plan`, `/list`) rather than feature-numbered; feature logic lives in `src/features/feat-NN-slug/`, which keeps the Stage 3 traceability.

### Directory Tree

```
plateful/
├── src/
│   ├── app/                                   # Next.js App Router — file-based routing
│   │   ├── layout.tsx                         # Root layout: html/body, fonts, providers (Query, i18n, toasts)
│   │   ├── manifest.ts                        # PWA web app manifest (installable, enables iOS Web Push)
│   │   ├── globals.css                        # Tailwind v4 entry + theme tokens (@theme)
│   │   ├── error.tsx / not-found.tsx          # Root error boundary + 404
│   │   ├── (public)/                          # Unauthenticated routes (no app shell)
│   │   │   ├── sign-in/ reset-password/       # FEAT-01 auth screens
│   │   │   ├── invite/[token]/                # FEAT-09 invitation acceptance
│   │   │   └── r/[code]/                      # FEAT-24 referral welcome
│   │   ├── (app)/                             # Authenticated household app — shell with bottom tab bar
│   │   │   ├── layout.tsx                     # App shell: tab bar (mobile) / sidebar (desktop), realtime provider
│   │   │   ├── setup/                         # FEAT-01 guided setup start + complete
│   │   │   ├── welcome/                       # FEAT-15 onboarding landing
│   │   │   ├── plan/                          # FEAT-03 plan view; nested swap/suggest/report/rate/vote/build
│   │   │   ├── list/                          # FEAT-06 grocery list; list/order FEAT-20
│   │   │   ├── pantry/                        # FEAT-05
│   │   │   ├── recipes/                       # FEAT-08 library + FEAT-10 import/new/edit
│   │   │   ├── votes/                         # FEAT-17 older-kid voting (Later)
│   │   │   ├── history/                       # FEAT-19
│   │   │   ├── check-in/                      # FEAT-25
│   │   │   ├── household/                     # FEAT-01 members, dietary rules, consent, budget; FEAT-18 remove member
│   │   │   ├── settings/                      # FEAT-01 hub, FEAT-16, FEAT-09, FEAT-21, FEAT-22 access record
│   │   │   ├── billing/                       # FEAT-14
│   │   │   ├── account/                       # FEAT-18 my account, export, delete household
│   │   │   ├── support/                       # FEAT-18 contact support
│   │   │   ├── invite-household/              # FEAT-24
│   │   │   └── handover/[requestId]/          # FEAT-09 hand-over acceptance
│   │   ├── (ops)/ops/support/                 # FEAT-22 operator console (operator role only)
│   │   └── api/
│   │       ├── inngest/route.ts               # Inngest function endpoint (all Automation specs that are scheduled/multi-step)
│   │       ├── webhooks/stripe/route.ts       # Payment webhooks (verify → enqueue)
│   │       └── v1/[resource]/route.ts         # JSON endpoints used by offline outbox replay
│   ├── features/                              # Feature business logic (NOT routing) — one folder per FEAT
│   │   └── feat-NN-{slug}/
│   │       ├── components/                    # Feature UI (client + server components)
│   │       ├── hooks/                         # TanStack Query hooks, Zustand slices
│   │       ├── actions/                       # Server Actions + Inngest functions (Automation specs)
│   │       ├── rules/                         # Pure business rules (Logic/Rule specs), shared client/server where safe
│   │       └── schemas.ts                     # Zod schemas for this feature's inputs
│   ├── server/                                # Server-only infrastructure ('server-only' import guard)
│   │   ├── db/                                # Drizzle client, schema/ (per-entity files), rls/ helpers
│   │   ├── auth/                              # Supabase server client, session + role guards
│   │   ├── inngest/                           # Inngest client, event catalogue, cron fan-out
│   │   ├── integrations/                      # One client module per external service (Integration specs)
│   │   ├── notifications/                     # Channel modules (email, push, in-app) + templates/
│   │   └── sync/                              # sync_seq allocation + broadcast after commit
│   ├── shared/                                # Cross-feature client-safe code
│   │   ├── components/ui/                     # shadcn/ui components (copied in, owned)
│   │   ├── components/                        # App-level shared components (TabBar, OfflineBanner, SafetyBadge)
│   │   ├── offline/                           # Service worker registration, IndexedDB outbox, replay
│   │   ├── locale/                            # Fixed unit-conversion table, aisle mapping, formatters
│   │   ├── lib/                               # utils, query client, supabase browser client
│   │   └── hooks/                             # useToast, useOnlineStatus, useRealtimeChannel
│   └── sw.ts                                  # Service worker source (app shell + plan/list caching)
├── drizzle/                                   # Generated SQL migrations (tables, RLS policies, functions)
├── content/                                   # Curated starter recipe batches (FEAT-08.SPEC-004 input)
├── tests/                                     # Unit (rules), integration (db + RLS), e2e (Playwright-class)
├── public/                                    # Icons, PWA assets
├── drizzle.config.ts
├── next.config.ts
├── package.json
└── tsconfig.json
```

### Feature-to-Directory Mapping

| Feature | Stage 3 Folder | Source Directory | Notes |
|---------|---------------|-----------------|-------|
| FEAT-01 (Household Setup & Member Profiles) | FEAT-01-household-setup-member-profiles/ | `src/features/feat-01-household-setup/`; routes `src/app/(public)/sign-in`, `reset-password`, `src/app/(app)/setup`, `household/`, `settings/` | Offline drafts via `src/shared/offline`; authorization rules (FEAT-01.SPEC-016) in `rules/` mirrored by RLS policies |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | FEAT-02-dietary-rules-allergy-safety-engine/ | `src/features/feat-02-safety-engine/`; route `src/app/(app)/plan/[date]/report` | Pure, server-only engine in `rules/`; allergen taxonomy data versioned in `rules/taxonomy/`; invoked by FEAT-03/04/08/10/17/19/23 |
| FEAT-03 (AI Weekly Dinner Plan Generation) | FEAT-03-ai-weekly-dinner-plan-generation/ | `src/features/feat-03-plan-generation/`; route `src/app/(app)/plan` | Generation is an Inngest function in `actions/`; Claude adapter in `src/server/integrations/claude.ts` |
| FEAT-04 (One-Tap Meal Swap) | FEAT-04-one-tap-meal-swap/ | `src/features/feat-04-meal-swap/`; routes `plan/[date]/swap`, `plan/[date]/suggest-swap`, `plan/suggestions` | Per-slot lock as conditional update in a Postgres function |
| FEAT-05 (Pantry-Aware Suggestions) | FEAT-05-pantry-aware-suggestions/ | `src/features/feat-05-pantry/`; route `src/app/(app)/pantry` | Offline adds through the shared outbox |
| FEAT-06 (Shared Grocery List) | FEAT-06-shared-grocery-list/ | `src/features/feat-06-grocery-list/`; route `src/app/(app)/list` | Merge rules (FEAT-06.SPEC-008) in `rules/`; sync via `src/server/sync` |
| FEAT-07 (Weekly Plan Ready Notification) | FEAT-07-weekly-plan-ready-notification/ | `src/features/feat-07-plan-ready/` | No screen; trigger in `actions/`, delivery through `src/server/notifications` |
| FEAT-08 (Recipe Library (Starter Recipes)) | FEAT-08-recipe-library-starter-recipes/ | `src/features/feat-08-recipe-library/`; routes `recipes`, `recipes/[recipeId]` | Seeding pipeline script in `actions/seed-batch.ts` reading `content/` |
| FEAT-09 (Household Invitations & Membership) | FEAT-09-household-invitations-membership/ | `src/features/feat-09-invitations/`; routes `(public)/invite/[token]`, `settings/invitations`, `settings/handover`, `handover/[requestId]`, `settings/leave` | Expiry as Inngest scheduled function |
| FEAT-10 (Recipe Import from Web Link) | FEAT-10-recipe-import-from-web-link/ | `src/features/feat-10-recipe-import/`; routes `recipes/import`, `recipes/import/[draftId]/review`, `recipes/new`, `recipes/[recipeId]/edit` | Extractor in `src/server/integrations/recipe-extractor.ts`; gated by feature flag |
| FEAT-11 (Leftover Rollover to Lunches) | FEAT-11-leftover-rollover-to-lunches/ | `src/features/feat-11-leftover-lunches/` | Card rendered inside `/plan`; computation is a step of the generation workflow |
| FEAT-12 (Meal Rating & Preference Learning) | FEAT-12-meal-rating-preference-learning/ | `src/features/feat-12-ratings/`; route `plan/[date]/rate` | Learned-dislike update as an Inngest event handler |
| FEAT-13 (Tonight's Dinner Reminder) | FEAT-13-tonights-dinner-reminder/ | `src/features/feat-13-tonight-nudge/` | No screen; timezone-bucketed cron fan-out |
| FEAT-14 (Subscription & Billing Management) | FEAT-14-subscription-billing-management/ | `src/features/feat-14-billing/`; routes `billing/*`; webhook `src/app/api/webhooks/stripe` | Billing state machine in `rules/billing-state.ts` |
| FEAT-15 (Member Onboarding) | FEAT-15-member-onboarding/ | `src/features/feat-15-onboarding/`; route `src/app/(app)/welcome` | Once-only rule in `rules/` |
| FEAT-16 (Units, Currency & Locale Configuration) | FEAT-16-units-currency-locale-configuration/ | `src/features/feat-16-locale/`; routes `settings/units`, `settings/aisles` | Conversion table in `src/shared/locale` (used by every feature) |
| FEAT-17 (Older-Kid Dinner Voting) | FEAT-17-older-kid-dinner-voting/ | `src/features/feat-17-dinner-voting/`; routes `votes/[roundId]`, `votes/[roundId]/outcome`, `plan/[date]/vote-setup` | Later phase; behind feature flag |
| FEAT-18 (Account & Data Management) | FEAT-18-account-data-management/ | `src/features/feat-18-account-data/`; routes `account/*`, `household/members/[memberId]/remove`, `support/new` | Export and deletion cascades as multi-step Inngest functions |
| FEAT-19 (Weekly Plan History) | FEAT-19-weekly-plan-history/ | `src/features/feat-19-plan-history/`; routes `history`, `history/[weekStart]` | Paginated server-rendered reads |
| FEAT-20 (Online Grocery Ordering Handoff) | FEAT-20-online-grocery-ordering-handoff/ | `src/features/feat-20-grocery-handoff/`; route `list/order` | Later phase; provider in `src/server/integrations/instacart.ts` |
| FEAT-21 (Family Calendar Sync) | FEAT-21-family-calendar-sync/ | `src/features/feat-21-calendar-sync/`; route `settings/calendar` | Later phase; provider in `src/server/integrations/google-calendar.ts` |
| FEAT-22 (Operator Read-Only Support Access) | FEAT-22-operator-read-only-support-access/ | `src/features/feat-22-support-access/`; routes `src/app/(ops)/ops/support/*`, `settings/support-access` | Read-only enforced by RLS + operator-scoped views |
| FEAT-23 (Manual Weekly Planning) | FEAT-23-manual-weekly-planning/ | `src/features/feat-23-manual-planning/`; routes `plan/build`, `plan/build/[date]/pick`, `plan/build/[date]/suggest` | Shares per-slot conditional writes with FEAT-04 |
| FEAT-24 (Invite Another Household) | FEAT-24-invite-another-household/ | `src/features/feat-24-household-referral/`; routes `invite-household`, `(public)/r/[code]` | Write-once attribution record |
| FEAT-25 (Weekly Waste & Spend Check-In) | FEAT-25-weekly-waste-spend-check-in/ | `src/features/feat-25-check-in/`; routes `check-in/[weekStart]`, `check-in/trend` | Lazily created weekly record plus scheduled cycle |

### Spec-Type-to-Location Mapping

| Spec Type | File Location Pattern | Example |
|-----------|----------------------|---------|
| Screen | `src/app/(app)/{route}/page.tsx` (+ `loading.tsx`, `error.tsx`) composing `src/features/feat-NN-{slug}/components/` | FEAT-06.SPEC-001 → `src/app/(app)/list/page.tsx` + `src/features/feat-06-grocery-list/components/GroceryList.tsx` |
| Automation | `src/features/feat-NN-{slug}/actions/{action}.ts` -- Server Actions for user-initiated work; Inngest functions (`*.fn.ts`) for scheduled or multi-step work, registered in `src/server/inngest/functions.ts` | FEAT-03.SPEC-003 → `src/features/feat-03-plan-generation/actions/generate-weekly-plan.fn.ts` |
| Logic/Rule | `src/features/feat-NN-{slug}/rules/{rule}.ts` (pure functions, unit-tested); authorization rules mirrored as RLS policies in `drizzle/` migrations | FEAT-02.SPEC-007 → `src/features/feat-02-safety-engine/rules/ingredient-completeness.ts` |
| Integration | `src/server/integrations/{service}.ts` -- one client module per external service, behind a typed interface | FEAT-14.SPEC-009 → `src/server/integrations/stripe.ts`; FEAT-03.SPEC-010 → `src/server/integrations/claude.ts` |
| Notification | `src/server/notifications/{channel}.ts` (email, push, in-app) + `src/server/notifications/templates/{notification}.tsx`; feature triggers in `actions/` call `notify()` | FEAT-07.SPEC-002 → `src/server/notifications/templates/plan-ready.tsx` sent via `push.ts` with `email.ts` fallback |

## 6. Data Layer Design

See `.n2b/architecture/database-schema.md` for the complete schema design.

**Migration strategy:** migration-based. Drizzle Kit generates versioned SQL migration files into `drizzle/` and they are applied in CI per environment (ADR-023). With 17 entities, 54 relationships, and RLS policies, database functions and Realtime settings that must evolve together with tables on a Supabase database holding children's-privacy-class data (ADR-003, ADR-004), every schema change needs to be reviewable, ordered and reproducible, which push-based auto-sync cannot guarantee.

## 7. API & Routing Architecture

### Route Map

Every Screen spec from the profile's Raw Spec Index (60) mapped to a URL. `[date]` is a plan night as an ISO date (`YYYY-MM-DD`); `[weekStart]` is the ISO date of the household week's first day.

| Screen Spec | URL Path | Parameters | Data Requirements |
|-------------|----------|------------|-------------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | /sign-in | `?mode=sign-up`, `?next=`, `?ref=` (referral code), `?invite=` | None before submit; creates Supabase Auth session |
| FEAT-01.SPEC-002 (Password Recovery) | /reset-password | `?step=request\|set` | Recovery token from email link |
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | /setup | -- | Household draft (name), setup progress |
| FEAT-01.SPEC-004 (Member List & Add Member) | /household/members | -- | Member Profiles (live via realtime), pending Invitations count |
| FEAT-01.SPEC-005 (Member Profile Detail) | /household/members/[memberId] | memberId | Member Profile, viewer role |
| FEAT-01.SPEC-006 (Dietary Rules Editor) | /household/members/[memberId]/dietary-rules | memberId | Dietary Rules for member, standard allergen list |
| FEAT-01.SPEC-007 (Parental Consent Confirmation) | /household/members/new/consent | `?draft=` (kid profile draft id) | Kid profile draft, organiser identity |
| FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | /household/budget-schedule | -- | Household budget, per-day time constraints, currency |
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | /setup/complete | -- | Household summary, Subscription tier |
| FEAT-01.SPEC-010 (Household Settings Hub) | /settings | -- | Household, viewer role, Subscription tier |
| FEAT-02.SPEC-001 (Report a Safety Concern) | /plan/[date]/report | date | Planned Meal, Recipe ingredients, open-report eligibility |
| FEAT-03.SPEC-001 (Weekly Plan View) | /plan | `?week=` (weekStart, defaults to current) | Weekly Plan + Planned Meals + Recipes, safety badges, budget total, pantry callouts, leftover lunches |
| FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) | /plan (rendered when Subscription tier = free) | `?week=` | Subscription tier, manual-plan state |
| FEAT-04.SPEC-001 (Meal Swap (Direct)) | /plan/[date]/swap | date | Planned Meal, ranked safe alternatives |
| FEAT-04.SPEC-002 (Suggest a Swap) | /plan/[date]/suggest-swap | date | Planned Meal, alternatives, existing open suggestion |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | /plan/suggestions | `?week=` | Open Swap Suggestions and pick suggestions for the week |
| FEAT-05.SPEC-001 (Pantry List & Item Entry) | /pantry | -- | Pantry Items (offline-cached) |
| FEAT-06.SPEC-001 (Grocery List) | /list | `?week=` | Grocery List + Items grouped by aisle, locale units (offline-cached, live) |
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | /recipes | `?q=`, `?badge=`, `?page=` | Recipe search results with per-household eligibility |
| FEAT-08.SPEC-002 (Recipe Detail View) | /recipes/[recipeId] | recipeId | Recipe, ingredients (locale-converted), safety badge |
| FEAT-09.SPEC-001 (Household Invitations Manager) | /settings/invitations | -- | Invitations (pending/expired), invite link |
| FEAT-09.SPEC-002 (Invitation Acceptance) | /invite/[token] | token | Invitation validity, household name, inviter first name |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | /settings/handover | -- | Eligible adult members, pending hand-over |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | /handover/[requestId] | requestId | Hand-over request, billing ownership notice |
| FEAT-09.SPEC-005 (Leave Household) | /settings/leave | -- | Viewer membership, consequences summary |
| FEAT-10.SPEC-001 (Import by Link) | /recipes/import | -- | Weekly import count (rate limit) |
| FEAT-10.SPEC-002 (Review Extracted Recipe) | /recipes/import/[draftId]/review | draftId | Extraction result, completeness flags, duplicate match |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | /recipes/new | -- | Unit list, locale |
| FEAT-10.SPEC-004 (Edit Imported Recipe) | /recipes/[recipeId]/edit | recipeId | Imported Recipe with version (for reject-with-refresh) |
| FEAT-11.SPEC-001 (Leftover Lunch Card) | /plan (card anchored `#lunch-[date]`) | `?week=` | Leftover-lunch Planned Meals linked to source dinners |
| FEAT-12.SPEC-001 (Post-Dinner Rating Prompt) | /plan/[date]/rate | date | Planned Meal, household members eligible to rate / proxy |
| FEAT-14.SPEC-001 (Plan Tier Overview) | /billing | -- | Subscription tier, billing_state, prices in household currency |
| FEAT-14.SPEC-002 (Upgrade to Paid) | /billing/upgrade | `?interval=month\|year` | Prices; creates Stripe Checkout session |
| FEAT-14.SPEC-003 (Billing & Payment Management) | /billing/manage | -- | Subscription, billing_state; links to Stripe Customer Portal |
| FEAT-14.SPEC-004 (Downgrade / Cancel) | /billing/cancel | -- | Current period end, refund eligibility |
| FEAT-15.SPEC-001 (Onboarding Landing) | /welcome | -- | Current plan + list summary, empty-household state |
| FEAT-16.SPEC-001 (Units & Currency Settings) | /settings/units | -- | Household unit system and currency |
| FEAT-16.SPEC-002 (Aisle Name Customization) | /settings/aisles | -- | Aisle mapping for household |
| FEAT-17.SPEC-001 (Dinner Vote Casting) | /votes/[roundId] | roundId | Voting round options, voter's existing vote |
| FEAT-17.SPEC-002 (Voting Round Setup) | /plan/[date]/vote-setup | date | Safe candidate recipes for the night |
| FEAT-17.SPEC-003 (Vote Outcome & Resolution) | /votes/[roundId]/outcome | roundId | Tally, resolution state |
| FEAT-18.SPEC-001 (Export Household Data) | /account/export | -- | Export job status, signed download URL |
| FEAT-18.SPEC-002 (Remove Member Profile) | /household/members/[memberId]/remove | memberId | Member Profile, consequences summary |
| FEAT-18.SPEC-003 (Delete Household) | /account/delete-household | -- | Household, Subscription state, confirmation |
| FEAT-18.SPEC-004 (My Account) | /account | -- | Viewer's account (email), own profile, notification preferences |
| FEAT-18.SPEC-005 (Contact Support) | /support/new | `?plannedMeal=` | Support Request categories |
| FEAT-19.SPEC-001 (Weekly Plan History Browse) | /history | `?cursor=` | Paginated archived Weekly Plans |
| FEAT-19.SPEC-002 (Past Week Detail View) | /history/[weekStart] | weekStart | Archived Weekly Plan + Grocery List |
| FEAT-20.SPEC-001 (Grocery Handoff Screen) | /list/order | -- | Unticked items snapshot, provider availability for market |
| FEAT-21.SPEC-001 (Calendar Connection Settings) | /settings/calendar | -- | Calendar connection state |
| FEAT-22.SPEC-001 (Support Request Queue) | /ops/support | `?status=` | Support Requests across households (operator only) |
| FEAT-22.SPEC-002 (Support Read-Only Household View) | /ops/support/[requestId]/household | requestId | Masked, read-only household projection; access session logged |
| FEAT-22.SPEC-003 (Household Support Access Record) | /settings/support-access | -- | Support access log entries for household (organiser) |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | /plan/build | `?week=` | Weekly Plan nights with picks, versions |
| FEAT-23.SPEC-002 (Pick / Change a Recipe) | /plan/build/[date]/pick | date, `?q=` | Eligible recipes (search), night version |
| FEAT-23.SPEC-003 (Suggest a Pick) | /plan/build/[date]/suggest | date | Eligible recipes, existing suggestion |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | /invite-household | -- | Viewer's personal referral link |
| FEAT-24.SPEC-002 (Referral Welcome Screen) | /r/[code] | code | Inviter first name only |
| FEAT-25.SPEC-001 (Weekly Check-In Card) | /check-in/[weekStart] | weekStart | Check-In record for week, household currency |
| FEAT-25.SPEC-002 (Check-In Trend View) | /check-in/trend | -- | Check-In history vs starting point |

### API Endpoint Convention

Mutations from screens use **Server Actions** co-located in `src/features/*/actions/`. Each takes a Zod-validated input that includes a client-generated `opId` (UUID idempotency key) and, for contended entities, the `expectedVersion` the client last saw. Each returns a discriminated result: `{ ok: true, data, syncSeq }` or `{ ok: false, code: 'CONFLICT' | 'FORBIDDEN' | 'VALIDATION' | 'RATE_LIMITED' | 'UNAVAILABLE', message, current? }`. `CONFLICT` carries the current server state so the UI can apply reject-with-refresh. The offline outbox replays the same operations over a small versioned JSON surface, `/api/v1/{resource}` with plural kebab-case resources (`/api/v1/grocery-list-items`, `/api/v1/pantry-items/{id}`). It uses `POST` to create, `PATCH` to update, `DELETE` to remove and `GET` to read, with an `Idempotency-Key` header, and responses reuse the result envelope as JSON. Webhooks live under `/api/webhooks/{provider}`, and Inngest under `/api/inngest`. There is no public third-party API in v1.

### Data Fetching Strategy

| Route Type | Fetching Approach | Rationale |
|-----------|-------------------|-----------|
| Live collaborative pages (`/plan`, `/list`, `/pantry`, `/household/members`) | Server Component renders the first paint; data is hydrated into TanStack Query, persisted to IndexedDB, and patched by Supabase Realtime `sync_seq` events | ASMP-22 (instant ticks, 2-second propagation) and ASMP-25 (offline viewing) need a client cache that survives connectivity loss; the Real-time signal (FEAT-03.SPEC-011, FEAT-06.SPEC-005) drives live patching |
| List/search pages (`/recipes`, `/history`, `/ops/support`) | Server Components with URL search params (`?q=`, `?cursor=`); streamed with `loading.tsx` skeletons; cursor pagination | ASMP-23 about-1-second search is served by one indexed Postgres query (ADR-012); history grows about 52 weeks per household per year (ASMP-24) |
| Detail pages (`/recipes/[recipeId]`, `/history/[weekStart]`, member detail) | Server Components fetching by id under RLS; dynamic rendering (no static caching of household data) | Household-private data (ASMP-26) must never be cached across users; details are read-mostly |
| Form submissions (setup, settings, billing, imports) | Server Actions with optimistic UI via TanStack Query mutations; offline-capable forms (setup drafts, pantry, list) enqueue into the IndexedDB outbox | FEAT-01.SPEC-013 requires drafts and offline queuing; reject-with-refresh rules need the server to return current state on conflict |
| Long-running operations (plan generation, recipe extraction, export) | Server Action emits an Inngest event and returns a job id; the UI subscribes to the household channel for completion, with a timed "taking longer than usual" state | ASMP-23 "explained wait"; FEAT-03.SPEC-010 and FEAT-10.SPEC-005 degradation contracts |

### Navigation Model

Mobile-first, one-thumb navigation (ASMP-29). A fixed **bottom tab bar** carries the five primary destinations: **Plan** (`/plan`), **List** (`/list`), **Recipes** (`/recipes`), **Pantry** (`/pantry`), **More** (`/settings`). On wide screens it becomes a left sidebar with the same items. Plan-night actions (swap, suggest, report, rate, vote setup) are nested under `/plan/[date]/…` and open as full-screen sheets on mobile, so the back gesture returns to the week. `/list` is the hub destination (profile Section 2: Shared Grocery List has 3+ inbound connections, from FEAT-03, FEAT-23 and FEAT-15): plan changes surface a "View list" action, and onboarding lands on `/welcome`, which links plan and list together. Settings-type features (units, aisles, invitations, hand-over, calendar, support access, billing, account) sit under **More**. Organiser-only entries are hidden for other roles per the Access Matrix. The operator console (`/ops/…`) is a separate route group with its own minimal shell and is never linked from household navigation. Deep links from notifications and emails point to the exact route (for example, a plan-ready push opens `/plan?week=…`, and a nudge opens `/plan#[date]`).

## 8. Integration Architecture

| Service | Purpose | Data Exchanged | Direction | Rate/Quota Notes | Failure-Mode Handling | Sandbox/Test Path |
|---------|---------|----------------|-----------|------------------|----------------------|-------------------|
| Anthropic Claude API | AI text/plan generation (External Touchpoint ASMP-30; FEAT-03.SPEC-010, FEAT-04.SPEC-007) and extraction fallback (FEAT-10.SPEC-005) -- ADR-011 | Out: household constraints without names or ages, aggregated ratings, pantry items, pre-verified candidate pool; in: structured candidate selections plus coverage signal. Children's-privacy-class, so zero-retention terms are required | Bidirectional (request/response) | Per-org tokens-per-minute limits; Sunday-evening burst throttled by Inngest concurrency (ADR-013); prompt caching for the shared prefix | Per FEAT-03.SPEC-010 Degradation Behavior: prior week stays visible with a Retry banner, a "This is taking longer than usual" note, late responses discarded, out-of-pool candidates dropped and short results treated as partial. Per FEAT-04.SPEC-007: Retry with the original meal unchanged, a note after 8 seconds, deterministic alternatives shown first | Separate API key per environment; recorded-response fixtures in CI; spike harness with synthetic households |
| OneSignal | Device-notification delivery (External Touchpoint ASMP-31; FEAT-07.SPEC-005, shared by FEAT-13.SPEC-002/004, FEAT-04.SPEC-006, FEAT-23 pick alerts) -- ADR-009 | Out: push subscription external id (member id), message title/body, deep link; no dietary data in payloads | Outbound (plus delivery receipts in) | Free tier; dispatch staggered within arrival slots | Per FEAT-07.SPEC-005 Degradation Behavior: offline devices receive queued push on reconnect; a push outage falls back to email (FEAT-07.SPEC-006); the plan's in-app availability never depends on delivery (XBR-12) | Separate OneSignal app per environment; test subscribers only in dev/staging |
| Resend | Transactional email (External Touchpoint ASMP-32; FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-14.SPEC-012, FEAT-18.SPEC-012) -- ADR-009 | Out: recipient email, templated content (recovery links, billing notices, export-ready links, safety reports to operator) | Outbound (plus bounce webhooks in) | Free tier 3,000 emails/month, then paid tier | Per FEAT-01.SPEC-017 Degradation Behavior: sends are queued (Inngest retries) and never block account creation, with non-disclosing timing. Per FEAT-02.SPEC-010: an operator-email outage leaves the Support Request visible in the Support View. Per FEAT-18.SPEC-012: in-app states are unaffected. Per FEAT-07.SPEC-006: an email outage sends nothing and shows no error | Test API key; dev environment sends only to allow-listed addresses |
| Stripe Billing | Payment processing (External Touchpoint ASMP-33; FEAT-14.SPEC-009) -- ADR-010 | Out: customer (organiser email), price, Checkout and Portal sessions; in: subscription and invoice events. Card data never touches Plateful | Bidirectional (API out, webhooks in) | API rate limits far above need; webhooks at-least-once and possibly out of order | Per FEAT-14.SPEC-009 Degradation Behavior: a processor outage disables Subscribe with a message and queues grace retries without shortening the 7-day window. Webhooks are verified, stored by event id, and processed idempotently (ADR-026) | Stripe test mode keys per environment; Stripe CLI webhook forwarding locally; test clocks for renewal/grace scenarios |
| Curated starter recipe content (editorial pipeline) | Recipe/food-content data (External Touchpoint ASMP-34; FEAT-08.SPEC-004) -- ADR-018 | In: batches of ingredient-complete recipes (no personal data) from the `content/` source into the database | Inbound (batch) | Batch-sized; no runtime quota | Per FEAT-08.SPEC-004 Degradation Behavior: a content-source problem has no household-visible impact; incomplete batches are held out by the completeness gate | Seed a reduced batch into dev/staging; completeness-gate unit tests |
| Supabase Realtime | Real-time data synchronisation (External Touchpoint ASMP-35; FEAT-03.SPEC-011, FEAT-06.SPEC-005) -- ADR-015 | Out/in: `{entity, id, sync_seq}` change notices on private per-household channels; payloads fetched under RLS | Bidirectional | Pro-plan concurrent-connection figure (~200 per landscape) to be validated by the FEAT-06 spike | Per FEAT-06.SPEC-005 Degradation Behavior: persistent offline banner, and every read and write stays usable and queued for sync. Per FEAT-03.SPEC-011: the plan stays viewable offline and approval needs a reconnect. `sync_seq` gaps trigger a re-fetch | Supabase local stack (CLI) for dev; separate project per environment |
| Recipe websites via the self-built extractor | Web-page recipe extraction (External Touchpoint ASMP-36; FEAT-10.SPEC-005) -- ADR-019 | Out: HTTP GET of the pasted URL (no cookies, no user data); in: page HTML, parsed to ingredients, steps and cook time | Outbound fetch | 30 imports per household per week (FEAT-10.SPEC-007); per-host concurrency cap; size and time limits | Per FEAT-10.SPEC-005 Degradation Behavior: progress indicator until timeout, then a failure path offering manual entry (FEAT-10.SPEC-003). SSRF protections block private addresses | Fixture HTML corpus of US/UK recipe pages in CI; feature flag off in production until the legal decision |
| Instacart Developer Platform (Later) | Online grocery ordering (External Touchpoint ASMP-37; FEAT-20.SPEC-002) -- ADR-020 | Out: snapshot of unticked list items; in: acceptance/rejection and item availability | Bidirectional | Partner terms via developer dashboard | Per FEAT-20.SPEC-002 Degradation Behavior: when down, nothing is sent and the list stays unchanged with Retry; slow responses show "Still working"; duplicate or late responses for an attempt are ignored | IDP development keys; handoff hidden outside US market |
| Google Calendar API (Later) | Family calendar (External Touchpoint ASMP-37; FEAT-21.SPEC-002) -- ADR-021 | Out: per-night entry (dish name and night only, FEAT-21.SPEC-005); in: sync outcome, token revocation | Bidirectional | Google API quotas well above one entry per planned meal | Per FEAT-21.SPEC-002 Degradation Behavior: sync failures are silent with per-night retries; a failed connect returns to Not Connected with the plan unaffected. The deletion cascade revokes tokens (FEAT-18.SPEC-008) | Google Cloud test project with test users before OAuth verification |
| Supabase Auth (architecture-introduced, ADR-029) | Identity, sessions, password recovery | Out/in: adult member email, password hash (held by Supabase), session tokens, custom claims (household id, role) | Bidirectional | Pro plan MAU allowance (50,000 free MAU per landscape) | Auth outage blocks sign-in; existing sessions keep working offline for cached plan/list viewing (ASMP-25); recovery emails go through Resend with queued retry | Supabase local stack; separate project per environment |
| Supabase Storage (architecture-introduced, ADR-008) | Household export files (FEAT-18.SPEC-006) | Out: generated export file; in: signed-URL download by organiser | Bidirectional | Pro plan 100GB included | Export job retries automatically (FEAT-18.SPEC-006); the user sees progress and never an indefinite wait | Local stack storage; per-environment buckets |
| Inngest (architecture-introduced, ADR-013) | Durable scheduled and multi-step workflows | Out: events with ids only (household id, plan id), never dietary details; in: function invocations to `/api/inngest` | Bidirectional | Free tier 25,000 runs/month; $75/month tier at growth | Step retries with backoff; failed runs alert through Sentry; idempotency keys prevent double sends (FEAT-07.SPEC-002) | Inngest Dev Server locally; branch environments for previews |
| Sentry (architecture-introduced, ADR-034) | Error tracking, cron/job monitoring, uptime checks | Out: error events and stack traces, with PII scrubbing and no session replay on household-data or operator screens | Outbound | Developer tier 5,000 errors/month; Team $26/month | If Sentry is down, the app is unaffected (fire-and-forget SDK) | Separate DSN per environment |
| PostHog (architecture-introduced, ADR-016) | Product analytics and feature flags | Out: events keyed by household and member ids only; in: feature-flag evaluations | Bidirectional | 1M events/month free | Flags default to safe values when PostHog is unreachable (for example, FEAT-10 import stays off) | Separate project per environment |

## 9. Design System Implementation Plan

**Design system posture:** None (`design_system_source: none`). This blueprint is design-agnostic: no design system is part of the package, and the downstream builder owns visual design. The builder should honour the design preferences recorded in the brief's Constraints, which reach this document through ASMP-29 ("big tap targets for one-handed use in a shop"; readable at larger OS text sizes; safety badges and ineligibility reasons "conveyed in words, not by color alone"). Styling and component decisions below are driven by product needs alone.

**Styling-system implementation approach:** Tailwind CSS v4 (ADR-005). The builder's visual tokens (colour, type scale, spacing, radius) go into a single `@theme` block in `src/app/globals.css` as CSS custom properties, so a later design system can be applied in one place. Product-driven constraints ship as tokens and component defaults, not visual design: a minimum 44px touch-target size on interactive components, rem-based typography (so it honours OS text-size settings), and a `SafetyBadge` component whose API requires a text label (it cannot render colour-only). Light/dark theming is left to the builder.

### Component Library Decision

**Selected: shadcn/ui (copy-in components) on Radix Primitives, styled with Tailwind CSS**, with components copied into `src/shared/components/ui/` and owned by the project (ADR-027).

Rationale from profile metrics: Interaction Complexity is Large (46 Automation specs; real-time and collaboration signals present) across 60 Screen specs. The landscape's component-layer note names recurring rich patterns: multi-step setup wizards (FEAT-01, 10 Screen specs), swap-suggestion review lists (FEAT-04), voting rounds (FEAT-17), and safety badges/disclaimers (FEAT-02). The screens also need bottom sheets, dialogs, toasts and accessible form controls. Hand-building accessible dialog, sheet, select and toast behaviour for 60 screens would repeat solved work. Radix primitives supply accessible behaviour (focus management, screen-reader semantics) that ASMP-29 depends on. Because the package is design-agnostic, the copy-in model is visually unopinionated where it matters: components are plain Tailwind-styled source files the builder restyles freely, with no third-party theme to override. shadcn/ui assumes React + Tailwind, which ADR-001 and ADR-005 satisfy (landscape Section 3 binding note).

Rejected component-layer candidates (from the landscape note): **Headless UI**, a viable React option with a smaller primitive set (no sheet or toast) than Radix, so more is hand-built. **Melt UI** is Svelte-only and incompatible with ADR-001. **Hand-built components only**, rejected on the interaction-complexity evidence above; it becomes the choice if the builder brings its own complete component library.

## 10. Shared Infrastructure Patterns

| Pattern | When Used | Approach | File Location | Dependencies |
|---------|-----------|----------|---------------|-------------|
| Layout system | Every route | Root layout (providers, fonts, PWA manifest). Route groups: `(public)` has a minimal centred shell, `(app)` has the household shell with tab bar or sidebar, an offline banner and the realtime channel provider, and `(ops)` has the operator shell. Plan-night actions render as nested routes shown as sheets on mobile | `src/app/layout.tsx`, `src/app/(public)/layout.tsx`, `src/app/(app)/layout.tsx`, `src/app/(ops)/layout.tsx` | Next.js App Router, Tailwind, shadcn/ui Sheet |
| Navigation | All authenticated screens | `TabBar` (mobile bottom, 5 items, thumb-reachable) and `SideNav` (desktop) built from one role-filtered nav config. Active state comes from `usePathname()` prefix match. Organiser-only items are hidden per Access Matrix; the server still enforces access | `src/shared/components/navigation/` (`nav-config.ts`, `TabBar.tsx`, `SideNav.tsx`) | Session role claims (ADR-029), next-intl for labels |
| Error handling | Route rendering errors, Server Action failures, background job failures | Per-segment `error.tsx` boundaries with a retry button and plain-language copy. Server Actions never throw to the client and return the typed result envelope (Section 7); `CONFLICT` renders the reject-with-refresh pattern ("This changed while you were editing" plus current state). Unexpected errors are captured to Sentry with PII scrubbing (ADR-034). Integration degradations follow each spec's Degradation Behavior copy | `src/app/**/error.tsx`, `src/shared/lib/result.ts`, `src/shared/components/ConflictNotice.tsx`, `src/server/observability.ts` | Sentry SDK, shadcn/ui Alert |
| Loading/empty states | Streaming server pages, pending mutations, long jobs, empty collections | `loading.tsx` skeletons per segment; optimistic updates for list, pantry and ratings (no spinners on the instant paths, per ASMP-22); `JobProgress` for generation, extraction and export with a timed "taking longer than usual" message; `EmptyState` with an explanation and a primary action (for example, the FEAT-15 empty household) | `src/app/**/loading.tsx`, `src/shared/components/{Skeleton,EmptyState,JobProgress,OfflineBanner}.tsx` | TanStack Query mutation state, Realtime job-completion events |
| Form handling | Setup wizard, settings, recipe entry/edit, billing, support | Controlled forms validated by a shared Zod schema (per feature `schemas.ts`), used both client-side for inline errors and server-side in the Server Action (ADR-028). Submission goes through a Server Action with `opId` and `expectedVersion`. Offline-capable forms persist drafts to the Zustand/IndexedDB store and enqueue on submit. After success, reset to server state; after a conflict, keep the user's input alongside the refreshed values | `src/features/*/schemas.ts`, `src/shared/components/form/`, `src/shared/offline/outbox.ts` | Zod, Zustand, TanStack Query, shadcn/ui form controls |
| Toast/notification | Action feedback (saved, queued offline, conflict, undo), and in-app inbox badge | shadcn/ui toast placed bottom-centre above the tab bar (thumb zone). Auto-dismiss after about 4 seconds for success; persistent until dismissed for errors and queued-offline states. Text always states the outcome in words (ASMP-29). The in-app notification inbox (feasibility cross-feature theme) is a database-backed list with a realtime badge, separate from toasts | `src/shared/hooks/useToast.ts`, `src/shared/components/ui/toast*`, `src/features/feat-07-plan-ready/components/Inbox.tsx` (shared inbox UI), `src/server/notifications/in-app.ts` | shadcn/ui, Supabase Realtime |

## 11. Authentication & Access Architecture

### Authentication & Identity

| Field | Value |
|-------|-------|
| Context | Authentication signal: Yes, complexity role-based -- FEAT-01.SPEC-001 (sign-up/sign-in), FEAT-01.SPEC-002 (password recovery), FEAT-01.SPEC-016 and FEAT-18.SPEC-011 (authorization rules). The Access Matrix defines 5 role rows: Organiser, Other Adult Member, young kid profile (no login), older kid (limited login, Later), Operator. Compliance/privacy: Yes (ASMP-26/27); feasibility FEAT-01: any auth vendor storing profile data "falls inside the children's-data processing boundary"; FEAT-18: household deletion signs out every member immediately; FEAT-22: read-only must be enforced server-side; FEAT-17 (Later): minors' limited-login model is an open question (feasibility Open Question 12) |
| Recommended | Supabase Auth (included in Supabase Pro, ADR-003): email + password sign-in with email-link recovery for adult members only, a custom-access-token hook adding `household_id` and `household_role` claims, and Postgres row-level security as the final enforcement layer. Operator accounts carry an `operator` app-role and require TOTP multi-factor authentication. Kid profiles have no credentials in v1 |
| Rationale | The landscape notes Supabase Auth is "free with the Supabase database if that Database option is selected; row-level-security integrates directly with Postgres role checks needed for kid-profile data restrictions" (50,000 free MAU). With the database already on Supabase, it adds no new processor to the children's-data boundary, and one policy language (RLS) enforces the Access Matrix across Server Actions, Realtime channels and Storage. Global session revocation supports FEAT-18.SPEC-008's immediate sign-out. Role logic is product-specific (organiser hand-over, Own-only rules), so it would be custom code under any provider |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Clerk | Free to 50,000 MRU; ~$1,000/month at 100K MAU | Low -- prebuilt components | High | Moderate -- hosted user store | Low | The database moves off Supabase (ADR-003 alternative) and prebuilt sign-in, invitation and profile UI is worth a separate identity vendor |
| Better Auth (self-hosted library) | Free; runs in the app | Medium -- team owns session hardening | High | Low | Mainstream plus security review | A legal review (feasibility Open Question 12) requires older kids' credentials and consent records to stay entirely inside Plateful's own database with no identity vendor |
| Auth.js (NextAuth) | Free, self-hosted | Medium -- manual role wiring | High | Low | Mainstream Next.js | Social sign-in (Google/Apple) becomes a requirement and the team wants framework-native providers without Supabase Auth |

### Session Model

- **Mechanism:** Supabase Auth sessions stored in cookies via Supabase's SSR helpers for Next.js. A short-lived JWT access token (1 hour) carries `sub`, `household_id`, `household_role` and, for staff, `app_role=operator`. A rotating refresh token renews it.
- **Refresh:** Next.js middleware refreshes the session on each request that has an expired access token. Refresh-token rotation with reuse detection is enabled. The client refreshes proactively when coming back online, before replaying the offline outbox.
- **Lifetime:** Assumption — 30-day inactivity timeout for household members (mobile users should not re-authenticate in the supermarket); 12-hour time-boxed sessions for operator accounts, which also need MFA. Upstream documents set no figures.
- **Where state lives:** Session state in Supabase Auth (server) plus cookies (client). Role claims are derived at token issue from `member_profiles`. Role changes (hand-over FEAT-09.SPEC-009, removal FEAT-18.SPEC-007) force a token refresh, and RLS re-reads membership from the database, so a stale token cannot outlive a role change. Household deletion (FEAT-18.SPEC-008) and member removal revoke all refresh tokens for affected users (global sign-out).
- **Offline:** A cached session lets the installed app open and show cached plan and list data offline. Queued writes replay only after a successful refresh. If authorization changed meanwhile (for example, hand-over while offline), the replay surfaces a retry/failed state and is never dropped silently (FEAT-01.SPEC-013).

### User Model Fields

Consistent with the Member Profile entity (7 fields, profile Section 5); the Schema Designer owns final column definitions.

- **Identity (Supabase `auth.users`, adults and operator only):** `id`, `email`, password hash (held by Supabase), `email_confirmed_at`, `last_sign_in_at`, MFA factors (operator), `app_metadata.app_role` (`operator` for staff, absent for household users).
- **Member profile (`member_profiles`):** `id`, `household_id`, `auth_user_id` (nullable; null for kid profiles), `display_name` (first name or nickname only for kids, ASMP-26), `member_type` (`adult` \| `kid`), `household_role` (`organiser` \| `adult` \| `kid`, exactly one organiser per household per XBR-15), `age_band` (kids only), `kid_login_enabled` (Later, FEAT-17; default false), `parental_consent_confirmed_at` + `parental_consent_confirmed_by` (kids; FEAT-01.SPEC-007), `onboarding_completed_at` (FEAT-15), `notification_preferences` (channel opt-ins, FEAT-07.SPEC-003), `version` (optimistic concurrency), `created_at`, `removed_at`.
- **Household-level identity data:** `households.organiser_member_id`, `households.timezone` (drives FEAT-13 and FEAT-03 schedules), `households.stripe_customer_id` (organiser-controlled billing).
- **Push subscriptions:** `push_subscriptions(member_id, onesignal_subscription_id, platform)` for adults only (kids and the operator are never push recipients, XBR-13).

### Protected Routes

Driving specs: FEAT-01.SPEC-001/002 (auth), FEAT-01.SPEC-016 (setup authorization), FEAT-06.SPEC-009, FEAT-09.SPEC-011, FEAT-12.SPEC-003, FEAT-14.SPEC-006, FEAT-17.SPEC-006, FEAT-18.SPEC-011, FEAT-22.SPEC-006/007 (authorization and visibility rules). Route guards in middleware are a first line only; every Server Action and query re-checks authorization, and RLS enforces it in the database.

| Route / Route Group | Access Requirement | Unauthenticated Experience |
|--------------------|--------------------|----------------------------|
| `/sign-in`, `/reset-password` | Public | Shown directly |
| `/invite/[token]` | Public (token-scoped; shows household name and inviter first name only) | Invitation acceptance screen with sign-in/sign-up (Access Matrix note) |
| `/r/[code]` | Public (inviter first name only, FEAT-24.SPEC-002) | Referral welcome page |
| `(app)` group: `/plan`, `/list`, `/pantry`, `/recipes/**`, `/history/**`, `/check-in/**`, `/welcome`, `/account`, `/support/new`, `/invite-household`, `/plan/[date]/rate`, `/plan/[date]/suggest-swap`, `/plan/build/[date]/suggest` | Authenticated household member (Organiser or Other Adult Member; Own-only actions enforced per action) | Redirect to `/sign-in?next=…` |
| `/setup/**`, `/household/**`, `/settings`, `/settings/units`, `/settings/aisles`, `/settings/invitations`, `/settings/handover`, `/settings/calendar`, `/settings/support-access`, `/plan/[date]/swap`, `/plan/suggestions`, `/plan/build`, `/plan/build/[date]/pick`, `/plan/[date]/vote-setup`, `/list/order`, `/account/export`, `/account/delete-household`, `/household/members/[memberId]/remove` | Role-gated (Organiser) for changes; Other Adult Member gets View where the Access Matrix grants View (e.g. Household Setup, Kid Profile Data) and is redirected to the view screen otherwise | Redirect to `/sign-in?next=…` |
| `/billing/**` | Role-gated (Organiser); Other Adult Member has None (Access Matrix) | Redirect to `/sign-in?next=…` |
| `/settings/leave`, `/handover/[requestId]` | Authenticated adult member (hand-over acceptance only by the named recipient) | Redirect to `/sign-in?next=…` |
| `/plan/[date]/report` | Authenticated adult member (Safety Reports: Organiser Full, Other Adult Member Own-only) | Redirect to `/sign-in?next=…` |
| `/votes/[roundId]`, `/votes/[roundId]/outcome` (Later) | Older-kid limited login (cast own vote) or Organiser; Other Adult Member View | Redirect to `/sign-in?next=…` |
| `(ops)` group: `/ops/support/**` | Role-gated (Operator, MFA); household view only while a Support Request is open for that household (FEAT-22.SPEC-006), read-only projection with kid and payment fields masked (FEAT-22.SPEC-007) | Hidden: returns 404 to anyone without the operator role |

### Role & Permission Mapping

| Role (Access Matrix) | Application Representation | Capability Access Summary | Enforcement Point |
|----------------------|---------------------------|---------------------------|-------------------|
| Maya (Organiser) | `member_profiles.household_role = 'organiser'`; JWT claim `household_role=organiser`; exactly one per household | Full on every capability group, including Billing, Household Invitations, Kid Profile Data and Account & Data; View on Support View (the access record) | Middleware route guard; Server Action authorization rules (`rules/authorization.ts` per feature); RLS policies on every household table |
| Sam (Other Adult Member) | `household_role = 'adult'`; claim `household_role=adult` | Full: Pantry Input, Grocery List, Recipe Library, Household Referrals, Waste Check-In. Own-only: Meal Swap and Manual Planning (suggestions), Ratings, Notification Prefs, Account & Data, Safety Reports. View: Household Setup, Weekly Plan (plus leftover eaten/skipped marks), Kid Profile Data, Dinner Voting. None: Household Invitations, Billing, Support View | Server Action rules plus RLS (`USING household_id = claim AND …` with per-action predicates); billing routes excluded at middleware |
| Jordan (young kid profile, no login — MVP) | `member_type = 'kid'`, `auth_user_id IS NULL`; no credentials, never a session | None on every group; data is managed through the Organiser's Household Setup access, and ratings are recorded by an adult as proxy (FEAT-12.SPEC-002) | Structural: no auth identity can exist; RLS blocks any direct access; kids are excluded from push/email recipients (XBR-13) |
| Jordan (older kid, limited login — Later) | `member_type = 'kid'`, `kid_login_enabled = true`, linked `auth_user_id`; claim `household_role=kid` (identity and consent model pending feasibility Open Question 12) | View: Weekly Plan, Recipe Library. Full: Grocery List (add/tick items only). Own-only: Ratings, Dinner Voting. None: everything else | Middleware allow-list of kid routes; RLS kid-role policies limited to list items, own votes and own ratings; feature-flagged until the consent model is resolved |
| Riley (Operator, support — from v1) | Supabase Auth user with `app_metadata.app_role = 'operator'` and MFA; not a household member | Full use of the read-only Support View. View: Household Setup, Weekly Plan, Pantry, Grocery List, Recipe Library, Ratings, Billing (tier only), Safety Reports. None: Kid Profile Data (except allergy details inside a specific safety report), Meal Swap, Manual Planning, Invitations, Referrals, Account & Data, Waste Check-In, Dinner Voting | `(ops)` route group guard; SECURITY DEFINER read-only views that return a masked projection only while an open Support Request exists; append-only access-session log (FEAT-22.SPEC-004); no write policies for the operator role on any household table |

## 12. Development Conventions

### Naming Conventions

| Category | Convention | Example |
|----------|-----------|---------|
| Files (components) | PascalCase `.tsx`, one component per file | `GroceryListItem.tsx` |
| Files (utilities) | kebab-case `.ts`; Inngest functions end `.fn.ts`; schemas in `schemas.ts` | `ingredient-completeness.ts`, `generate-weekly-plan.fn.ts` |
| Components | PascalCase, noun-first, feature-prefixed only when ambiguous | `SafetyBadge`, `SwapAlternativeList` |
| Functions | camelCase, verb-first; Server Actions end in `Action`; rules are `check*`/`is*`/`compute*` | `applyMealSwapAction()`, `checkCandidateSafety()` |
| CSS classes | Tailwind utilities; shared variants via `cva()` in the component file; no global class names except in `globals.css` | `className={cn("min-h-11 px-4", className)}` |
| Database tables | snake_case plural; columns snake_case; foreign keys `{entity}_id` | `grocery_list_items.grocery_list_id` |
| Routes | kebab-case, lowercase, product nouns; dynamic segments camelCase in brackets | `/plan/[date]/suggest-swap`, `/household/members/[memberId]` |

### Component Structure

Order inside a component file: (1) `'use client'` directive only when needed (default to Server Components), (2) imports in the order below, (3) exported props type, (4) the component as a named function export, (5) small private subcomponents and helpers below it. No default exports except Next.js route files (`page.tsx`, `layout.tsx`), which the framework requires.

```tsx
'use client'
import { useState } from 'react'
import { Button } from '@/shared/components/ui/button'
import { useGroceryList } from '@/features/feat-06-grocery-list/hooks/use-grocery-list'

export type GroceryListProps = { weekStart: string }

export function GroceryList({ weekStart }: GroceryListProps) { /* … */ }
```

### Import Ordering

Groups separated by a blank line and enforced by the linter: (1) React/Next.js, (2) third-party packages, (3) `@/server/*` (server files only), (4) `@/shared/*`, (5) `@/features/*`, (6) relative imports, (7) type-only imports last.

```ts
import { revalidatePath } from 'next/cache'

import { z } from 'zod'

import { db } from '@/server/db'
import { requireRole } from '@/server/auth'

import { ok, conflict } from '@/shared/lib/result'

import { checkCandidateSafety } from '@/features/feat-02-safety-engine/rules/check-candidate'

import { toPlannedMeal } from './mappers'

import type { PlannedMeal } from '@/server/db/schema'
```

### TypeScript Usage

`strict: true` plus `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` (ADR-031). Use `type` aliases by default and `interface` only for extendable public contracts such as provider interfaces (`interface PlanGenerator`). `any` is banned by lint; use `unknown` plus Zod parsing at every trust boundary (Server Action input, webhook payload, LLM output, IndexedDB replay). Database types are inferred from the Drizzle schema (`typeof plannedMeals.$inferSelect`) and never hand-duplicated. `import 'server-only'` guards every module under `src/server/`.

```ts
const parsed = PlanGenerationResult.safeParse(llmJson) // unknown → typed, or a handled failure
if (!parsed.success) return partialResult('invalid_ai_output')
```

### Path Aliases

Defined in `tsconfig.json` `paths`:

```json
{
  "@/app/*": ["./src/app/*"],
  "@/features/*": ["./src/features/*"],
  "@/server/*": ["./src/server/*"],
  "@/shared/*": ["./src/shared/*"],
  "@/tests/*": ["./tests/*"]
}
```

## 13. Deployment & Environments

### Hosting & Environments

| Field | Value |
|-------|-------|
| Context | Frontend and backend: Next.js (ADR-001, ADR-002); jobs on Inngest (ADR-013), which calls back into the app; database, auth, realtime and storage on Supabase (ADR-003). Section 2: "No specific uptime number was stated"; several thousand households; sub-$100/month pre-revenue budget; US and UK users (BRIEF.md Geography) |
| Recommended | Vercel Pro (1-2 seats) for the Next.js app, with production, a long-lived staging environment, and per-pull-request preview deployments. Functions are deployed in the US East region co-located with the Supabase project, with maximum function duration raised for the generation and extraction steps |
| Rationale | The landscape rates Vercel the "first-party host for Next.js/Turbopack; per-environment preview deployments fit a dev/staging/prod topology" at $20/seat/month. It is the lowest-operations option for a serverless Next.js app whose durable work already runs through Inngest (so Render's always-on workers are not needed). US East co-location with the database minimises query latency for the list hot path. UK users get static assets from the CDN, and the offline-first client (ASMP-25) masks transatlantic latency on writes |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Render | Starter web service $25/month; workers extra | Low-Medium -- containers and cron | High | Low-Moderate -- Docker deploys | Mainstream | Jobs move to BullMQ workers (ADR-013 alternative) or the realtime layer moves to self-hosted Socket.io, both of which need always-on processes |
| Netlify | Pro $20/month, no seat limits | Low | High | Moderate | Mainstream | Team size grows and per-seat Vercel pricing becomes a material cost, or Vercel-specific features are not used |
| Fly.io | Two shared CPUs + 512MB at $5-10/month | Medium -- explicit machine/region config | High | Low-Moderate -- container images | Container operations | UK users become the majority and the app should run in two regions near users, together with a database read replica |

### CI/CD & Delivery

| Field | Value |
|-------|-------|
| Context | Hosting: Vercel (ADR-032); 196 specs including 55 Logic/Rule specs (unit-testable rules) and a safety engine rated Hard that needs a labelled regression suite (feasibility FEAT-02 mitigation); migration-based schema changes (ADR-023); children's-privacy-class data (ASMP-27) |
| Recommended | GitHub Actions: on every pull request run type-check, lint, unit tests (rules, including the allergen regression suite as a required check), integration tests against a local Supabase stack (RLS policies tested per role), and Drizzle migration dry-run. On merge to `main`, apply migrations to staging, then deploy staging; production is promoted by a tagged release that applies production migrations and then promotes the Vercel build |
| Rationale | The landscape rates GitHub Actions "the 2026 default" with zero-config use when code is on GitHub, at $0.006/Linux minute with included free minutes, well inside the budget. It integrates with Vercel previews, the Supabase CLI and the Inngest branch environments used by the topology below. Making the allergen regression suite a merge-blocking check turns FEAT-02's highest risk into a delivery gate |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| GitLab CI | CE free; GitLab.com tiers | Medium | High | Zero when self-hosted | Mainstream | Source moves to GitLab, or compliance requires self-hosted runners and registry |
| CircleCI | Comparable per-minute pricing | Low-Medium | High | Moderate -- orbs/config | Mainstream | Test-suite duration (the 55 rule specs plus RLS integration tests) becomes a bottleneck that native test splitting fixes |
| Buildkite | Usage-based orchestration; self-hosted agents | Medium-High | Very high | Low-Moderate | DevOps | A compliance review requires that build agents handling production data snapshots run only on the team's own infrastructure |

**Environment topology:**
- **Local dev:** a developer machine running the Supabase local stack (Postgres, Auth, Realtime, Storage in containers), the Inngest Dev Server, Stripe CLI webhook forwarding, and test keys for Claude, Resend and OneSignal. Seeded with a reduced starter recipe batch and synthetic households, never production data.
- **Preview (per pull request):** Vercel preview deployment against the **staging** Supabase project and an Inngest branch environment. Test-mode Stripe; email sends restricted to allow-listed addresses. Used for review and QA of individual changes.
- **Staging (`main` branch):** a long-lived Vercel environment plus a separate Supabase project, Inngest environment, OneSignal app, PostHog project and Sentry environment. Migrations are applied here first. Test-mode Stripe with test clocks for renewal and grace scenarios. It mirrors production configuration and uses synthetic data only (children's-privacy posture, ASMP-27).
- **Production (tagged release):** Vercel production plus the production Supabase project (Pro, daily backups, point-in-time recovery if budget allows), live Stripe, production keys. Secrets are held per environment in Vercel environment variables and GitHub Actions environment secrets. Production secrets are scoped to the production environment with required-reviewer protection, and the Supabase service-role key is never exposed to the client.

**Infrastructure as code:** Not warranted at launch. The footprint is five managed SaaS projects configured through their dashboards and CLIs, and database state is already code (Drizzle migrations, including RLS). A short `docs/runbook/environments.md` checklist records every dashboard setting per environment. Growth trigger: adopt Terraform (or the provider CLIs scripted in CI) when a second region, a second production deployment (for example a UK data-residency split), or a team beyond about 5 engineers makes configuration drift a real risk.

### Observability & Operations

| Field | Value |
|-------|-------|
| Context | Section 2: no stated uptime number, but the list and plan must keep working offline; background processing: Yes (7 scheduled automations, 46 Automation specs); payment webhooks (FEAT-14.SPEC-009); feasibility risks: Sunday-evening bursts, webhook ordering, and observability tools capturing masked kid data from operator screens (FEAT-22); Compliance/privacy: Yes |
| Recommended | Sentry (Developer tier at launch, Team $26/month at growth) for frontend and backend error tracking with PII scrubbing (no session replay on household-data or `(ops)` screens), Sentry Crons check-ins for each scheduled Inngest function, and uptime monitoring of the production health endpoint. Vercel runtime logs, Supabase logs and the Inngest run dashboard provide logging and job visibility. Alerts go to email/Slack on error spikes, missed cron check-ins and failed webhook processing |
| Rationale | The landscape describes Sentry as organising around "what broke in my code", with a free Developer tier and SDKs for the selected stack. That fits a team-owned codebase with 46 automations. Cron check-ins answer the specific operational risk that a missed weekly generation or 4:00 pm nudge fails silently. Using platform-native logs rather than a separate log vendor avoids another processor of household data and keeps cost inside SC-16 |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Better Stack | Volume-based, no per-seat fees | Low-Medium | High | Moderate | Mainstream | The team wants errors, centralised logs, uptime and on-call incident management in one vendor once someone carries a pager |
| Datadog | From $15/host/month, compounding modules | Medium | Very high | Higher | Observability specialist | Distributed tracing across a split backend (ADR-002 alternatives) and realtime paths becomes necessary to debug latency against ASMP-22 |
| New Relic | Usage-based; free tier | Medium | Very high | Moderate-High (NRQL) | Observability specialist | Same trigger as Datadog, with a preference for ingest-based pricing |

### Indicative Cost Model

Order-of-magnitude only; figures come from the landscape's pricing evidence and must be verified by the implementing team. "Launch" is pre-revenue through early paid households; "Growth tier" is several thousand households with a meaningful paid share (ASMP-24).

| Component | Launch (order of magnitude) | Growth tier |
|-----------|-----------------------------|-------------|
| Vercel Pro (hosting) | ~$20-40/month (1-2 seats at $20/seat) | ~$40-100/month (seats plus bandwidth beyond 1TB at $40/100GB) |
| Supabase Pro (database, auth, realtime, storage) | ~$25/month | ~$25-100/month (compute add-on, egress overage at $0.125/GB) |
| Anthropic Claude API (paid tier only) | ~$0-50/month (few paid households; ~$2/$10 per 1M tokens with prompt caching) | ~$100-500/month usage-scaled (one plan plus a few swaps per paid household per week) |
| Inngest (background jobs) | Free tier (25,000 runs/month) | ~$75/month (first paid tier) |
| Resend (email) | Free tier (3,000 emails/month) | ~$20/month order of magnitude (paid tier; landscape lists Postmark's comparable tier at $19.95/month) |
| OneSignal (Web Push) | Free tier | Free tier to ~$100/month if paid features are adopted |
| Stripe Billing (payments) | 2.9% + $0.30 per charge + 0.7% billing volume (usage-only, $0 fixed) | Same percentage rates; ~$620/month annual Billing tier only if volume makes it cheaper |
| Sentry (observability) | Free Developer tier (5,000 errors/month) | ~$26/month (Team) |
| PostHog (analytics, flags) | Free tier (1M events/month) | ~$0-300/month usage-scaled beyond 1M events |
| GitHub Actions (CI/CD) | Free included minutes | ~$10-30/month at $0.006/Linux minute |
| Curated starter recipe content | One-time editorial/licensing cost; $0/month recurring | $0/month recurring (editorial upkeep is staff time) |
| Later: Google Calendar API / Instacart IDP | $0 (not built at launch) | $0 API cost for Google Calendar; Instacart on partner commercial terms |

Fixed pre-revenue platform spend is about $45-65/month (Vercel + Supabase, everything else on free tiers), inside the sub-$100/month budget (SC-16). AI spend starts only with paid households, per BRIEF.md "The free tier gets no AI."

### Local Development Setup

Prerequisites: Node.js (current LTS), a container runtime for the Supabase local stack, the Supabase CLI, the Stripe CLI, and the Inngest Dev Server (run through `npx`). Path from clone to running: clone → install dependencies → copy `.env.example` to `.env.local` (test-mode keys only) → `supabase start` → apply Drizzle migrations and run the seed (reduced starter recipes plus synthetic households) → start the Next.js dev server and the Inngest Dev Server → `stripe listen --forward-to localhost:3000/api/webhooks/stripe`. Local substitutes: the Supabase local stack replaces the managed database, auth, realtime and storage; Stripe test mode with test clocks; the Inngest Dev Server; Claude, Resend and OneSignal with separate development keys (Resend restricted to allow-listed addresses); PostHog and Sentry disabled locally by default. Web Push and PWA install testing need HTTPS: use the preview deployment or a local HTTPS tunnel.

## 14. Decision Log

| ID | Category | Decision | Rationale (one-line) | Profile Driver |
|----|----------|----------|---------------------|---------------|
| ADR-001 | Frontend Framework | Next.js (React, App Router) delivered as an installable PWA | Largest real-time/PWA/offline ecosystem; installability enables iOS Web Push; Server Components keep 60-screen bundle small | Interaction Complexity: Large; 60 Screen specs; Offline signal: Yes; BRIEF "mobile-first responsive web app... no native apps in v1" |
| ADR-002 | Backend / API Layer | Next.js Route Handlers + Server Actions (Node) with framework-independent domain modules; durable work offloaded to Inngest | Avoids a second deployable at Medium scale; shared TypeScript types across 17 entities; job platform absorbs the 46-automation load | Scale: Medium; 15 Integration + 46 Automation specs; Background processing signal: Yes |
| ADR-003 | Database | Supabase (managed Postgres), Pro plan, single region | Relational fit plus RLS for household/kid isolation; bundles Auth, Realtime, Storage at $25/month | Data Complexity: Medium (17 entities, 54 relationships); Compliance/privacy signal: Yes; SC-16 sub-$100/month budget |
| ADR-004 | ORM / Data Access | Drizzle ORM + Drizzle Kit SQL migrations via Supabase pooler | Code-first types, serverless-friendly, plain SQL carries RLS policies and functions in the same migrations | 17 entities, 54 relationships; Compliance/privacy signal: Yes (RLS) |
| ADR-005 | CSS / Styling | Tailwind CSS v4 | Zero runtime, Server Component support, tokenised tap-target/type constraints; prerequisite for shadcn/ui | 60 Screen specs; ASMP-29 one-thumb accessibility |
| ADR-006 | State Management | TanStack Query (persisted, offline-paused mutations) + Zustand (UI state, offline outbox) | Server-cache patched by realtime events plus durable outbox covers live sync and offline queuing | Real-time: Yes; Offline: Yes; Collaboration/concurrency on 16 of 17 entities |
| ADR-007 | Build Tooling | Next.js bundled pipeline (Turbopack dev, default production build) | No profile driver justifies overriding the framework pipeline | Frontend Framework selection (ADR-001); Scale: Medium (25 features) |
| ADR-008 | File & Object Storage | Supabase Storage private bucket, signed expiring URLs, scheduled purge | Single low-volume export use case; no extra processor inside the children's-data boundary | Import/export signal: Yes (FEAT-18.SPEC-001, FEAT-18.SPEC-006); File upload: No |
| ADR-009 | Email & Messaging Delivery | Resend (email) + OneSignal (Web Push), fallback and dedupe in app code | Free tiers cover launch; fastest Web Push setup; delivery rules testable in one module | Notifications signal: Yes (20 Notification specs); ASMP-31, ASMP-32 |
| ADR-010 | Payments & Billing | Stripe Billing (hosted Checkout + Customer Portal), app-owned grace state machine | Lowest per-charge cost; hosted UI speeds the three-month revenue path | Payments/billing signal: Yes (FEAT-14.SPEC-001..012); ASMP-33 |
| ADR-011 | AI & Intelligent Behavior | Anthropic Claude API (Sonnet-tier) behind a provider interface, prompt caching, pre-filtered verified pool; deterministic-first swap alternatives | Structured output plus caching keeps per-household cost small; adapter plus spike keep choice reversible | AI/ML behavior signal: Yes (FEAT-03.SPEC-010, FEAT-04.SPEC-007); BRIEF Cost "one weekly plan plus a few swaps"; ASMP-23 |
| ADR-012 | Search | Postgres full-text search (tsvector + GIN) joined with precomputed safety attributes | Simple-filter complexity; one indexed query meets the 1-second target with eligibility | Search signal: Yes (simple filter, FEAT-08.SPEC-001); ASMP-23 |
| ADR-013 | Background Jobs & Scheduling | Inngest durable functions with timezone-bucketed cron fan-out, per-household concurrency keys, provider throttling | Durable multi-step workflows without workers; enforces one-run-per-household and smooths Sunday bursts | Background processing signal: Yes (7 scheduled automations); 46 Automation specs |
| ADR-014 | Caching & Performance | Service worker + IndexedDB offline layer; Postgres precomputation instead of a server cache | Offline is device-side; avoids stale safety-verdict cache while meeting latency | Offline signal: Yes (FEAT-01.SPEC-013, FEAT-06.SPEC-008); Scale hints ASMP-22/23 |
| ADR-015 | Real-time & Collaboration | Supabase Realtime private household Broadcast channels with server-assigned per-household sync_seq | Bundled transport; server ordering removes client-clock skew and makes delivery gap-detectable | Real-time signal: Yes (FEAT-03.SPEC-011, FEAT-06.SPEC-005); Collaboration/concurrency on 16 of 17 entities |
| ADR-016 | Analytics & Product Telemetry | PostHog Cloud with id-only event properties, restricted session replay, feature flags | 1M free events; flags gate FEAT-10 and Later features; self-host exit path | Scale hints signal: Yes (ASMP-24 several thousand households); ASMP-26 privacy |
| ADR-017 | Internationalization | next-intl formatting (en-US/en-GB) + shared fixed-factor conversion module | Formatting and conversion need, English only; conversion works offline | Internationalization signal: Yes (FEAT-16.SPEC-001, FEAT-16.SPEC-002, FEAT-16.SPEC-004); ASMP-28 |
| ADR-018 | Recipe & Food Content Data | Curated/licensed owned starter set + in-house versioned allergen taxonomy with regression suite | Highest control over fail-closed completeness; no recurring fee or storage-licence risk | Product-mandated ASMP-34; FEAT-02 verdict Hard; SC-16 budget |
| ADR-019 | Web Page Recipe Extraction | Hybrid TypeScript JSON-LD/Microdata parser + Claude fallback, SSRF-hardened, feature-flagged | Structured-first keeps AI cost low; single Node runtime; flag decouples legal decision | Product-mandated ASMP-36; FEAT-10.SPEC-005; Import/export signal: Yes |
| ADR-020 | Online Grocery Ordering Integration | Instacart Developer Platform (US, Later) behind a provider interface; UK pending partner spike | Only self-serve option matching FEAT-20.SPEC-002; interface admits a UK partner later | Product-mandated ASMP-37 (Later); FEAT-20 verdict Research-spike recommended |
| ADR-021 | Family Calendar Integration | Google Calendar API direct (Later) behind a provider interface | Free and sufficient for narrow one-calendar scope; unified API only when demand shows | Product-mandated ASMP-37 (Later); FEAT-21.SPEC-002 |
| ADR-022 | Project Structure | Single Next.js package; product-oriented URLs in route groups; feature logic in src/features/feat-NN-slug | One deployable at Medium scale; preserves FEAT traceability without numbered URLs | 25 features, 196 specs; Scale: Medium |
| ADR-023 | Data Migration Strategy | Migration-based (Drizzle Kit generated SQL, applied in CI per environment) | RLS policies, functions and tables must evolve reviewably and reproducibly together | 17 entities, 54 relationships; Compliance/privacy signal: Yes |
| ADR-024 | API & Routing | Product-noun URL map (60 Screen specs) with plan-night actions nested under /plan/[date]; Server Actions + versioned /api/v1 outbox surface with idempotency keys | One-thumb navigation depth; offline replay needs idempotent HTTP endpoints | 60 Screen specs; hub screen Shared Grocery List (3 inbound); Offline signal: Yes |
| ADR-025 | API & Routing | Mixed data fetching: RSC first paint + persisted TanStack Query + realtime patches for live pages; RSC + URL params for lists; Server Actions for forms; event + job id for long operations | Matches each route type to instant, offline, 1-second and explained-wait targets | ASMP-22, ASMP-23, ASMP-25; Real-time signal: Yes |
| ADR-026 | Integration Architecture | Webhook ingestion: verify signature → persist by provider event id → emit Inngest event → idempotent, ordering-tolerant state-machine processing | At-least-once, out-of-order processor events must not corrupt billing_state or grace timing | FEAT-14.SPEC-009 Inbound Events; Payments/billing signal: Yes |
| ADR-027 | CSS / Styling | Component layer: shadcn/ui on Radix Primitives, copied into src/shared/components/ui | Accessible dialogs/sheets/toasts for rich patterns; unopinionated for a design-agnostic package | Interaction Complexity: Large; 60 Screen specs; design_system_source: none; ASMP-29 |
| ADR-028 | Shared Infrastructure Patterns | Shared Zod schemas validate forms client- and server-side; typed result envelope with CONFLICT carrying current state. Deviation: Zod is outside the landscape because no validation-library area exists in the registry; chosen as the de facto TypeScript schema library that also validates LLM output and webhook payloads | One schema per input across client, server, outbox replay and AI output parsing | Collaboration/concurrency on 16 of 17 entities (reject-with-refresh); 15 Integration specs |
| ADR-029 | Authentication & Identity | Supabase Auth (email + password, recovery) with household_role custom claims, RLS enforcement, operator MFA; no kid credentials in v1 | No new processor in the children's-data boundary; one policy language across API, Realtime and Storage | Authentication signal: Yes (role-based; FEAT-01.SPEC-001, FEAT-01.SPEC-016); Access Matrix: 5 role rows; ASMP-27 |
| ADR-030 | Authentication & Identity | Session model: cookie-based SSR sessions, 1-hour JWT, rotating refresh tokens, 30-day member inactivity / 12-hour operator timebox, global revocation on removal or deletion | Keeps shoppers signed in offline while role changes and deletions take effect immediately | FEAT-18.SPEC-008 immediate sign-out; FEAT-01.SPEC-013 offline replay; Offline signal: Yes |
| ADR-031 | Development Conventions | Strict TypeScript (strict + noUncheckedIndexedAccess + exactOptionalPropertyTypes), no any, Zod at trust boundaries | Fail-closed safety logic and offline replay need compile-time exhaustiveness | FEAT-02 verdict Hard; 55 Logic/Rule specs |
| ADR-032 | Hosting & Environments | Vercel Pro (production, staging, per-PR previews), functions co-located with Supabase in US East | First-party Next.js host; serverless fits since durable work runs on Inngest | Scale: Medium; ASMP-24 several thousand households; no stated uptime number (BRIEF) |
| ADR-033 | CI/CD & Delivery | GitHub Actions with allergen regression suite and per-role RLS tests as required checks; staging-then-tagged-production promotion with migrations | Default CI at negligible cost; turns the Hard safety risk into a merge gate | 55 Logic/Rule specs; FEAT-02 verdict Hard; ASMP-27 |
| ADR-034 | Observability & Operations | Sentry (errors, cron check-ins, uptime) with PII scrubbing and restricted replay; platform-native logs | Catches silent job failures cheaply without adding a log processor for household data | Background processing signal: Yes (7 scheduled automations); Compliance/privacy signal: Yes |
