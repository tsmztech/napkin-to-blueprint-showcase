---
document_type: technical-profile
produced_by: profile-analyst
status: final
stage: 4
created: 2026-09-28
project_name: Plateful
---

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
