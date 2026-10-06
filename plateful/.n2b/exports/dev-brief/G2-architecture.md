# Part G2 — Architecture: Recommendation & Alternatives

This part carries the four architecture documents in reading order: evidence, option space, feasibility, then decisions. The recommendation is the default build; alternatives are documented for every decision area, with `Choose instead when` conditions, for this team to weigh.

## The Evidence — Technical Profile


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


## The Option Space — Technology Landscape


# Technology Landscape

## 1. Research Scope

| Decision Area | Activated By |
|---|---|
| Frontend Framework | Always active |
| Backend / API Layer | Always active |
| Database | Always active |
| ORM / Data Access | Always active |
| CSS / Styling | Always active |
| State Management | Always active |
| Build Tooling | Always active |
| Authentication & Identity | Always active |
| Hosting & Environments | Always active |
| CI/CD & Delivery | Always active |
| Observability & Operations | Always active |
| File & Object Storage | Import/export (Present = Yes; FEAT-10.SPEC-001 Import by Link, FEAT-18.SPEC-001 Export Household Data, FEAT-18.SPEC-006 Export Generation Processing) |
| Email & Messaging Delivery | Notifications (email/push/SMS) (Present = Yes; 20 Notification specs, e.g. FEAT-07.SPEC-002 Plan-Ready Notification Message, FEAT-13.SPEC-002 Tonight's Dinner Nudge Message) |
| Payments & Billing | Payments/billing (Present = Yes; FEAT-14.SPEC-001 through FEAT-14.SPEC-012, Subscription & Billing Management) |
| AI & Intelligent Behavior | AI/ML behavior (Present = Yes; FEAT-03.SPEC-010 AI Plan Generation Capability Integration, FEAT-04.SPEC-007 Swap Alternatives Generation, FEAT-12.SPEC-004 Preference Weighting & Tier-Gating Rule) |
| Search | Search (Present = Yes; FEAT-08.SPEC-001 Recipe Library Browse & Search, complexity: simple filter) |
| Background Jobs & Scheduling | Background processing (Present = Yes; FEAT-03.SPEC-003 Scheduled Weekly Plan Generation, FEAT-07.SPEC-001 Plan-Ready Notification Trigger, FEAT-13.SPEC-001 Tonight's Nudge Trigger) · Import/export (Present = Yes; FEAT-18.SPEC-006 Export Generation Processing) · Notifications (email/push/SMS) (Present = Yes; 20 Notification specs) |
| Caching & Performance | Scale hints (Present = Yes; ASMP-22, ASMP-23, ASMP-24 -- instant list interactions, sub-minute plan generation, several thousand households) · Offline (Present = Yes; FEAT-01.SPEC-013 Setup Draft Persistence & Offline Queuing, FEAT-06.SPEC-008 Offline Conflict Resolution Rules) |
| Real-time & Collaboration | Real-time (Present = Yes; FEAT-03.SPEC-011 Real-Time Plan Sync Integration, FEAT-06.SPEC-005 Live Grocery List Sync) · Collaboration/concurrency (Present = Yes; last-write-wins/reject-with-refresh rules across 16 of 17 entities' Contention notes) |
| Analytics & Product Telemetry | Scale hints (Present = Yes; ASMP-24 several thousand households in year one, staying equally responsive as the base grows) |
| Internationalization | Internationalization (Present = Yes; FEAT-16.SPEC-001 Units & Currency Settings, FEAT-16.SPEC-002 Aisle Name Customization, FEAT-16.SPEC-004 Cross-Feature Value Conversion Rule) |
| Recipe & Food Content Data | Product-mandated -- ASMP-30 to ASMP-37 provenance chain, ASMP-34: "Recipe/food-content data capability -- Required to seed the starter recipe library at launch with complete ingredient data the allergy check can verify; without it, a brand-new household would have to rely entirely on manually imported recipes before any plan could be generated." (assumptions-constraints.md, Dependencies), tied to BRIEF.md's "Recipe sources: a starter recipe library, plus saving recipes from any website by pasting a link." |
| Web Page Recipe Extraction | Product-mandated -- ASMP-36: "Web-page recipe extraction capability (v1) -- Required for Recipe Import from Web Link (FEAT-10) to read a recipe's ingredients and steps from a pasted link; without it, households could still add recipes by manual entry. Whether importing from other sites is legally acceptable, and in what form, remains a brief open question to settle before v1 (BRIEF.md, Open Questions)." (assumptions-constraints.md, Dependencies) |
| Online Grocery Ordering Integration | Product-mandated -- ASMP-37 (first capability): "Online grocery-ordering and family-calendar capabilities (Later) -- Required only for Online Grocery Ordering Handoff (FEAT-20) and Family Calendar Sync (FEAT-21); the core product works fully without them." (assumptions-constraints.md, Dependencies), tied to BRIEF.md's "Online grocery ordering (e.g. Instacart, Tesco): desirable later, not v1." |
| Family Calendar Integration | Product-mandated -- ASMP-37 (second capability, same citation as above), tied to BRIEF.md's "Family calendar: a nice-to-have for showing dinner on the family calendar, not v1." |

## 2. Decision Area Landscapes

### Frontend Framework

Serves a mobile-first responsive web app used one-handed in a shop and on a laptop for setup (BRIEF.md, Devices/platforms: "a mobile-first responsive web app... no native apps in v1"), with Large interaction complexity (46 Automation specs, real-time and collaboration signals present per profile Section 4) and offline-tolerant screens (FEAT-01.SPEC-013, FEAT-06.SPEC-008).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Next.js (React) | Meta-framework | Strong SSR/SSG for responsive web, largest ecosystem for real-time client updates and PWA/offline tooling; 78% of new React apps use it | Open source; framework is free, hosting cost varies by platform | Low -- default entry point, deepest job-market and tooling support | Very mature; React ecosystem lock-in is low (portable component model), Next.js-specific APIs add moderate framework lock-in | https://www.intuz.com/best-frontend-frameworks/ (accessed 2026-09-28) |
| Nuxt 3 (Vue) | Meta-framework | Production-ready SSR/SSG meta-framework, 200+ module ecosystem, high developer-satisfaction-to-capability ratio | Open source; free | Low-Medium -- smaller talent pool than React but mature tooling | Mature; Vue's smaller ecosystem than React raises specialist-hire risk | https://www.brilworks.com/blog/javascript-web-frameworks-comparison/ (accessed 2026-09-28) |
| SvelteKit | Meta-framework | Smallest bundle sizes and highest raw perceived performance of the compared frameworks -- favorable for one-handed, low-signal mobile use | Open source; free | Medium -- smallest ecosystem of the three, fewer prebuilt integrations | Production-ready but 6.9% professional-developer adoption vs React's 46.9%, raising future-hire risk | https://www.mgsoftware.nl/en/tools/best-frontend-frameworks (accessed 2026-09-28) |
| Remix (React) | Meta-framework | React-based, nested routing and web-standards-first data loading; strong for form-heavy, progressively-enhanced flows | Open source; free | Low-Medium -- React ecosystem reuse, smaller dedicated community than Next.js | Mature (merged into React Router v7 lineage); lower lock-in via standard web APIs | https://www.coderio.com/blog/software-development/guide-frontend-frameworks-2026/ (accessed 2026-09-28) |
| Astro (React/Vue islands) | Meta-framework (islands architecture) | Content-first with islands of interactivity; strong for the mostly-static marketing/onboarding surface, weaker fit for the app's dense interactive screens | Open source; free | Medium -- islands model requires explicit hydration boundaries for interactive screens | Mature for content sites; smaller track record for interaction-heavy SPAs like this product's plan/grocery-list screens | https://midrocket.com/en/guides/best-frontend-frameworks/ (accessed 2026-09-28) |

### Backend / API Layer

Serves 15 Integration specs and 46 Automation specs (background jobs, scheduled generation, webhook-driven billing and payment events) requiring a Node/TypeScript-capable API layer that can share types with the frontend and support real-time channels.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| NestJS (on Node.js) | Framework | TypeScript-first, opinionated module/controller/service architecture that scales with team size and the product's 15 Integration + 46 Automation specs | Open source; free | Low-Medium -- structured but has a learning curve versus minimal frameworks | Mature, large ecosystem; can run on Express or Fastify adapters, low lock-in to either | https://dev.to/ihor_ostin/nestjs-vs-fastify-vs-express-which-backend-wins-in-2026-2ep2 (accessed 2026-09-28) |
| Fastify | Framework | Schema-driven serialization and 2-3x Express throughput -- useful for the "instant" grocery-list and swap responsiveness targets (ASMP-22) | Open source; free | Low -- lighter-weight, less structure imposed than NestJS | Mature, actively maintained; minimal proprietary surface | https://www.index.dev/skill-vs-skill/backend-nestjs-vs-expressjs-vs-fastify (accessed 2026-09-28) |
| Express.js | Framework | Simple, unopinionated routing; largest install base but weaker built-in structure for a 25-feature, 196-spec product | Open source; free | Low -- minimal setup, but more manual wiring for validation/DI | Extremely mature ("more npm downloads than any other server framework"), largely legacy-driven adoption in 2026 | https://tech-insider.org/fastify-vs-express-vs-nestjs-2026/ (accessed 2026-09-28) |
| Next.js Route Handlers / Server Actions | Framework-native backend layer | Co-located with the frontend framework if Next.js is chosen; avoids a second deployable for a medium-scale API surface | Included with the frontend framework; free | Low for simple CRUD, Medium as Integration-spec count (15) and background-job count grow | Mature; tightly coupled to Next.js, raising migration cost if the frontend framework changes | https://dev.to/nayankyada/nextjs-hosting-cost-in-2026-vercel-vs-netlify-vs-railway-vs-vps-431a (accessed 2026-09-28) |
| Django (Python) | Framework | Batteries-included ORM, admin, and auth; strong fit for data-model-dense products (17 entities, 54 relationships) but splits the stack from a TypeScript frontend | Open source; free | Medium -- separate language/runtime from the frontend, no shared types | Very mature; low framework lock-in, high migration cost only if switching languages entirely | https://quartzdevs.com/resources/best-backend-frameworks-2026-top-server-side-tools (accessed 2026-09-28) |

### Database

Serves 17 entities and 54 inter-entity relationships (profile Section 2) with dense relational structure, 20 cross-feature business rules, and household-scoped multi-tenant access patterns (role-based authorization across Organiser/Other Adult Member/kid profiles).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Neon (Serverless Postgres) | Managed service | Relational engine fits the 17-entity, 54-relationship graph; serverless scale-to-zero suits several-thousand-household, variable-traffic profile (ASMP-24) | Usage-based: $0.106/CU-hour (Launch), $0.222/CU-hour (Scale), storage $0.35/GB-month | Low -- standard Postgres wire protocol, works with any Postgres driver/ORM | Mature (Databricks-owned since May 2025); low lock-in via standard Postgres compatibility | https://www.bytebase.com/blog/postgres-hosting-options-pricing-comparison/ (accessed 2026-09-28) |
| Supabase (Postgres) | Managed service (BaaS) | Postgres plus built-in Auth, Realtime, and Storage -- directly overlaps this product's Real-time & Collaboration and Auth needs | Pro plan $25/month incl. 8GB storage, 250GB egress; overage $0.125/GB | Low -- integrated SDKs for auth/realtime/storage reduce cross-service wiring | Mature; moderate lock-in if adopting its bundled Auth/Realtime/Storage beyond the core database | https://dev.to/philip_mcclarence_2ef9475/best-postgresql-hosting-in-2026-rds-vs-supabase-vs-neon-vs-self-hosted-5fkp (accessed 2026-09-28) |
| Amazon RDS for PostgreSQL | Managed service | Full control over instance sizing and Multi-AZ; fits teams standardizing on AWS for Hosting & Environments | $30-140+/month once Multi-AZ and storage are added | Medium -- more manual provisioning (VPC, IAM, parameter groups) than serverless competitors | Extremely mature; low data-model lock-in (standard Postgres), moderate operational lock-in to AWS | https://dreamlit.ai/blog/top-10-managed-postgres-providers (accessed 2026-09-28) |
| PlanetScale for Postgres | Managed service | Resource-based pricing pro-rated to the millisecond; newer Postgres offering (2025) alongside its established MySQL platform | From $5/month (single node, 1/16 vCPU/512MB); 3-node HA from $15/month | Low -- standard Postgres wire protocol | Newer to Postgres specifically (MySQL platform is mature); low data lock-in via Postgres compatibility | https://selfhost.dev/blog/managed-postgresql-comparison-2026/ (accessed 2026-09-28) |
| Self-hosted PostgreSQL (e.g., on a VM or container platform) | Self-hosted | Full control and no vendor usage ceiling; requires the team to own backups, HA, and patching for a product processing children's data under compliance obligations (ASMP-27) | Infrastructure cost only (compute + storage), no vendor markup | High -- team owns provisioning, backups, failover, patching, and monitoring | Very mature engine; zero data-portability lock-in, but full operational ownership | knowledge-based -- self-hosting cost/effort is a general operational pattern rather than a single vendor's pricing page; no single authoritative pricing URL applies |

### ORM / Data Access

Serves a dense relational schema (17 entities, 54 relationships, 20 cross-feature business rules) that needs migration tooling capable of tracking schema evolution across 25 features.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Prisma ORM (v7) | ORM | Schema-first modeling suits a 17-entity domain; Prisma 7's TS/WASM query compiler removed the prior Rust-engine overhead (up to 3.4x faster on large result sets) | Open source core; Prisma Postgres/Accelerate add-ons priced separately | Low -- generated client, Prisma Migrate, and Prisma Studio reduce hand-written SQL and migration tooling gaps | Mature, large ecosystem; moderate lock-in to the `.prisma` schema DSL and generated client | https://makerkit.dev/blog/tutorials/drizzle-vs-prisma (accessed 2026-09-28) |
| Drizzle ORM | ORM / query builder | Code-first TypeScript schema with inferred types (no generation step); lighter runtime overhead, serverless-friendly for scale-to-zero database options | Open source; free | Medium -- Drizzle Kit migration tooling is less polished than Prisma Migrate for complex schema changes | Newer but fast-growing (recently overtook Prisma in npm downloads); low lock-in, generates plain SQL | https://www.bytebase.com/blog/drizzle-vs-prisma/ (accessed 2026-09-28) |
| TypeORM | ORM | Decorator-based, supports both Active Record and Data Mapper patterns; long-standing option for TypeScript/Node relational data | Open source; free | Medium -- decorator-heavy API, historically less type-safety than Prisma/Drizzle | Mature but slower recent innovation pace than Prisma/Drizzle | https://encore.dev/articles/prisma-vs-drizzle-vs-typeorm (accessed 2026-09-28) |
| SQLAlchemy | ORM | Mature Python ORM with strong relational modeling -- only pairs with a Python backend (e.g., Django/FastAPI), not a Node backend | Open source; free | Medium -- requires a Python backend choice to be paired correctly (see Section 3 sanity rule) | Very mature; low lock-in, broad database dialect support | https://quartzdevs.com/resources/best-backend-frameworks-2026-top-server-side-tools (accessed 2026-09-28) |

### CSS / Styling

Serves a mobile-first, one-handed, large-tap-target UI (ASMP-29) with badges and ineligibility reasons conveyed in words not color alone -- accessibility-sensitive styling across 60 Screen specs.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Tailwind CSS v4 | Utility-class framework | Dominant choice (~70% of new projects); zero runtime cost, full Server Component support, pairs with all mainstream frameworks researched above | Open source; free | Low -- largest ecosystem, fastest build times, lowest onboarding barrier | Very mature; low lock-in (utility classes are portable, no proprietary runtime) | https://vibetown.pro/articles/tailwind-css-vs-css-in-js-which-styling-approach-works-best-for-vibe-coders/69780eb759cc9309469500ed (accessed 2026-09-28) |
| vanilla-extract | Zero-runtime CSS-in-JS | Type-safe, build-time static CSS generation in TypeScript; most "CSS-like" of the zero-runtime tools, giving fine control for the design system's tokens | Open source; free | Medium -- more setup than Tailwind, requires TypeScript-based style authoring | Mature, actively maintained; low runtime lock-in (outputs static CSS) | https://www.pkgpulse.com/guides/vanilla-extract-vs-panda-css-vs-tailwind-2026 (accessed 2026-09-28) |
| Panda CSS | Zero-runtime CSS-in-JS | Combines CSS-in-JS developer experience with static generation -- JSX style props compiled to utility classes | Open source; free | Medium -- newer tool, smaller ecosystem than Tailwind | Newer entrant (part of the Chakra UI team's toolset); low runtime lock-in | https://kanopylabs.com/blog/panda-css-vs-tailwind-v4-vs-vanilla-extract (accessed 2026-09-28) |
| CSS Modules (vanilla CSS) | Native styling | Framework-agnostic, no additional dependency; scoped class names without a utility-class learning curve | Free (native to all frameworks researched) | Low -- built into virtually every frontend framework's tooling | Extremely mature, zero lock-in | https://www.kunalganglani.com/blog/tailwind-vs-css-modules-2026 (accessed 2026-09-28) |

Component-layer note (guide Section 6): interaction complexity is Large (46 automations, real-time/collaboration signals present) with recurring rich patterns -- multi-step setup wizards (FEAT-01), swap-suggestion review lists (FEAT-04), voting rounds (FEAT-17), and safety badges/disclaimers (FEAT-02) -- which justifies evaluating a component layer alongside the styling system. No user-supplied design system is referenced in the profile (design_system_source not asserted as user-supplied here). Candidate component-layer approaches, framework-dependent per Section 5's binding facts: shadcn/ui (copy-in components, assumes React + Tailwind), Radix Primitives (headless, React-only), Headless UI (headless, React and Vue), Melt UI (headless, Svelte-only).

### State Management

Serves real-time plan/grocery-list sync (FEAT-03.SPEC-011, FEAT-06.SPEC-005) alongside client-side UI state (multi-step setup wizard, swap review) -- a mixed server-state/client-state profile.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| TanStack Query + Zustand | Server-cache library + client store | The documented 2026 default pairing: TanStack Query owns remote/server state (plan, grocery list, sync), Zustand owns UI/client state (wizard steps, modals) | Both open source; free | Low -- widely documented pattern, minimal boilerplate | Mature, React/Vue/Svelte adapters exist for both; low lock-in (plain JS stores, standard fetch semantics) | https://dev.to/iamsaadmehmood/state-management-in-2026-redux-vs-context-vs-tanstack-query-1b0b (accessed 2026-09-28) |
| Redux Toolkit + RTK Query | Integrated state framework | Single cohesive store for both server and client state with shared DevTools; more structure for a team preferring one framework over the TanStack+Zustand pairing | Open source; free | Medium -- more boilerplate and conceptual overhead than the Zustand/TanStack pairing | Very mature; framework-agnostic core but RTK Query couples server-state caching to the Redux store (moderate lock-in) | https://thamizhelango.medium.com/the-ultimate-guide-to-react-state-management-zustand-vs-redux-vs-redux-toolkit-vs-rtk-query-vs-08b5655020f4 (accessed 2026-09-28) |
| Jotai | Atomic state library | Granular atom-based client state, useful for fine-grained UI state (e.g., per-field validation in setup screens) | Open source; free | Low-Medium -- different mental model (atoms) than store-based approaches | Mature, growing adoption; low lock-in, React-focused (Section 5 binding: SWR/Redux Toolkit are React-only, Jotai likewise) | https://xcodx.io/blog/libraries/best-react-state-management-libraries (accessed 2026-09-28) |
| Pinia | Store library | Vue's official state library -- only pairs with a Vue/Nuxt frontend choice | Open source; free | Low -- first-party Vue integration | Mature, officially maintained; framework-bound (Section 5 sanity rule: Pinia is Vue-only) | https://theroadtoenterprise.com/blog/zustand-vs-redux-toolkit (accessed 2026-09-28) |

### Build Tooling

Serves the bundled pipeline of whichever meta-framework is chosen (Frontend Framework area) -- build speed matters for a 25-feature, 196-spec codebase with frequent iteration.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Vite 8 (with Rolldown) | Bundler/dev server | Default for non-Next.js meta-frameworks (Nuxt, SvelteKit); Rust-based Rolldown engine gives up to 13x faster production builds | Open source; free | Low -- fast dev-server startup (5.8x faster than Webpack), minimal config | Very mature, healthiest 2026 community sentiment among the options researched | https://reintech.io/blog/javascript-build-tools-comparison-2026 (accessed 2026-09-28) |
| Turbopack | Bundler (Next.js-native) | Purpose-built for Next.js, reached stable status in 2026; 9.5x faster incremental builds than Webpack in dev | Included with Next.js; free | Low for Next.js projects -- bundled by default | Mature for dev builds; as of the researched sources, production builds still route through a separate pipeline, a Next.js-specific dependency | https://techsy.io/en/blog/turbopack-vs-webpack-vs-vite (accessed 2026-09-28) |
| Rspack | Bundler | Rust-based, near-instant dev startup, positioned as a drop-in Webpack replacement (v2.0) | Open source; free | Medium -- newer, smaller plugin ecosystem than Vite/Webpack | Newer (2.0 milestone in 2026); low lock-in as a Webpack-compatible tool | https://www.kunalganglani.com/blog/vite-turbopack-rspack-benchmark (accessed 2026-09-28) |
| Webpack 5 | Bundler | Broadest legacy plugin ecosystem; fits teams needing deep, non-standard build customization | Open source; free | High -- more configuration overhead than Vite/Turbopack for a greenfield project | Extremely mature; large plugin ecosystem lowers migration risk for edge-case build needs | https://www.eneskaymaz.com/en/blog/webpack-vs-vite-vs-turbopack-2026 (accessed 2026-09-28) |

### Authentication & Identity

Serves role-based access (Organiser/Other Adult Member/kid profiles) with parental-consent gating for kid profiles (FEAT-01.SPEC-007) and strict data-minimality/compliance requirements for children's data (ASMP-26, ASMP-27).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Clerk | Managed identity provider | Prebuilt React/Next.js components speed household-setup and invitation-acceptance screens; role/session model supports the multi-role household structure | Free to 50,000 MRU; ~$1,000/month at 100K MAU | Low -- drop-in UI components, best-documented Next.js integration among the options researched | Mature, fast-growing; moderate lock-in (hosted user store, migration requires user-data export) | https://www.iloveblogs.blog/post/nextjs-authentication-comparison-2026 (accessed 2026-09-28) |
| Supabase Auth | Managed identity provider (BaaS) | Free with the Supabase database if that Database option is selected; row-level-security integrates directly with Postgres role checks needed for kid-profile data restrictions | 50,000 free MAU; ~$162/month at 100K MAU on Supabase Pro | Low if Supabase is already the database choice -- otherwise adds a second vendor relationship | Mature; lock-in is moderate, tied to the Supabase platform | https://www.buildmvpfast.com/api-costs/authentication (accessed 2026-09-28) |
| Auth0 | Managed identity provider | Enterprise-grade protocol coverage (SSO, SCIM); more than this product's v1 household-role model requires, but available for future B2B-style needs | Free tier to 25,000 MAU; starts at $35+/month beyond that, enterprise plans in the thousands | Medium -- more configuration surface than Clerk for a simple household-role model | Very mature; moderate-to-high lock-in via Auth0's rules/actions extensibility model | https://www.buildmvpfast.com/blog/best-auth-providers-2026-clerk-supabase-comparison (accessed 2026-09-28) |
| Better Auth | Open-source library (self-hosted) | Self-hosted, framework-agnostic auth library; full control over the user/session model needed for parental-consent gating and kid-profile restrictions | Open source; free (self-hosted, team bears infrastructure cost) | Medium -- team owns session storage, flows, and security hardening instead of a hosted provider | Newer but the most-installed auth library by weekly downloads in 2026 (6.41M/week); low lock-in, self-hosted and open source | https://www.iloveblogs.blog/post/nextjs-authentication-comparison-2026 (accessed 2026-09-28) |
| NextAuth.js (Auth.js) | Open-source library | Framework-native (Next.js) session/auth handling with OAuth provider support; free and self-hosted | Open source; free | Medium -- more manual wiring for role-based access than a managed provider's admin UI | Mature (5.81M weekly downloads); low lock-in, self-hosted | https://www.iloveblogs.blog/post/nextjs-authentication-comparison-2026 (accessed 2026-09-28) |

### Hosting & Environments

Serves several thousand households in year one (ASMP-24) with dev/staging/prod environment needs and a mobile-first web delivery target with no specific uptime commitment stated (BRIEF.md, Performance/availability).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Vercel | Managed hosting platform | First-party host for Next.js/Turbopack; per-environment preview deployments fit a dev/staging/prod topology | Pro $20/seat/month; free tier 100GB bandwidth, Pro raises to 1TB with $40/100GB overage | Low -- zero-config deploys for the frontend-framework candidates researched | Very mature; moderate lock-in (platform-specific edge functions, ISR) though standard Next.js output is portable | https://gautamkhorana.com/blog/cloud-hosting-2026-vercel-netlify-cloudflare-render/ (accessed 2026-09-28) |
| Render | Managed hosting platform (PaaS) | Persistent Node containers with no cold starts -- fits background-job workers and the "instant" grocery-list responsiveness target (ASMP-22) better than pure serverless | Starter web service $25/month | Low-Medium -- broader than static/edge hosting, supports long-running processes and cron | Mature; low-moderate lock-in, standard container/Docker deploys | https://dev.to/nayankyada/nextjs-hosting-cost-in-2026-vercel-vs-netlify-vs-railway-vs-vps-431a (accessed 2026-09-28) |
| Fly.io | Managed hosting platform (edge compute) | Global edge deployment; cheapest entry point, suited to a household-scoped, geographically distributed (US/UK) user base | Two shared CPUs + 512MB RAM at $5-10/month | Medium -- more ops-hands-on than Vercel/Netlify (explicit machine/region config) | Mature; low-moderate lock-in via fly.toml config, standard container images | https://f3fundit.com/micro-saas-hosting-infrastructure-vercel-vs-railway-vs-render-vs-fly-io-2026/ (accessed 2026-09-28) |
| Netlify | Managed hosting platform | Comparable DX to Vercel with lower platform lock-in; September 2025 pricing overhaul removed seat limits on Pro | Pro $20/month, no seat limits | Low -- similar zero-config deploy flow to Vercel | Mature; native image CDN still in beta per the researched source, a gap versus Vercel's image optimization | https://gautamkhorana.com/blog/cloud-hosting-2026-vercel-netlify-cloudflare-render/ (accessed 2026-09-28) |
| AWS (ECS/Amplify + RDS) | Cloud infrastructure platform | Full infrastructure control across compute, database, and storage under one account -- suited to teams standardizing all Section 3/4 services on a single cloud vendor | Pay-as-you-go compute/storage; RDS alone runs $30-140+/month before app compute | High -- most manual provisioning and IaC investment of the options researched | Extremely mature; low data-portability lock-in but high operational/IaC lock-in to AWS-specific services | https://dreamlit.ai/blog/top-10-managed-postgres-providers (accessed 2026-09-28) |

### CI/CD & Delivery

Serves a 25-feature, 196-spec codebase needing automated checks and promotion across dev/staging/prod before each of the 46 automation and 15 integration specs ship.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| GitHub Actions | Managed CI/CD (SaaS) | Zero-config integration if source is hosted on GitHub; ecosystem breadth for the frontend/backend stack researched above | $0.006/Linux minute since the January 2026 rate cut; free minutes included on most plans | Low -- default choice when code already lives on GitHub | Very mature, the 2026 default for most teams per the researched sources | https://devopsboys.com/blog/github-actions-vs-gitlab-ci-vs-circleci-2026 (accessed 2026-09-28) |
| GitLab CI | Managed/self-hosted CI/CD | Full infrastructure ownership option (self-hosted runners, registry, artifacts) -- relevant given the product's children's-data compliance posture (ASMP-27) | GitLab CE is free and open source; GitLab.com tiers priced separately | Medium -- more setup than GitHub Actions unless already on GitLab | Very mature; zero vendor lock-in when self-hosted | https://devopsboys.com/blog/github-actions-vs-gitlab-ci-vs-circleci-2026 (accessed 2026-09-28) |
| CircleCI | Managed CI/CD (SaaS) | Native test-splitting and layer caching without extra plugin configuration; relevant if the 46-automation-spec test suite becomes a build-time bottleneck | Comparable per-minute pricing to GitHub Actions on its Medium resource class | Low-Medium -- requires a second CI account and config format distinct from GitHub-native workflows | Mature; moderate lock-in to CircleCI's config/orb ecosystem | https://www.kunalganglani.com/blog/github-actions-vs-circleci (accessed 2026-09-28) |
| Buildkite | Hybrid CI/CD (self-hosted agents, managed orchestration) | Agents run on the team's own infrastructure while Buildkite orchestrates -- useful if compliance requirements (ASMP-27) demand build agents stay off third-party compute | Usage-based orchestration pricing; agent compute is self-hosted | Medium-High -- team provisions and maintains its own build agents | Mature in the CI/CD space; low lock-in for build execution, moderate for the orchestration layer | https://labhub.hopto.org/blog/culture/2026-05-16-cicd-platforms-2026-github-actions-gitlab-ci-circleci-buildkite-earthly-dagger-deep-dive?lang=en (accessed 2026-09-28) |

### Observability & Operations

Serves a product with scheduled background jobs (7 Automation specs triggering on schedule per profile Section 3), real-time sync paths, and payment-processing webhooks -- failure modes that need error tracking and alerting, not just uptime pings.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Sentry | Developer-first error tracking / APM | Organizes around "what broke in my code" -- strong error grouping and session/frontend replay for a 60-Screen-spec UI | Free Developer tier (5,000 errors/mo, 1 user); Team $26/month (unlimited users, 50,000 errors); Business $80/month | Low -- SDKs for the frontend/backend frameworks researched above | Very mature; low-moderate lock-in, standard error-tracking data model | https://betterstack.com/community/comparisons/datadog-vs-sentry/ (accessed 2026-09-28) |
| Datadog | Full-stack observability platform | Broadest commercial platform -- infrastructure, APM, logs, RUM in one place, useful once the background-job and real-time surfaces need unified tracing | From $15/host/month, cost compounds across infra/APM/logs modules | Medium -- broader setup surface than a pure error tracker | Very mature; higher lock-in given the breadth of proprietary modules adopted | https://betterstack.com/community/comparisons/datadog-vs-sentry/ (accessed 2026-09-28) |
| Better Stack | Unified observability (logs, errors, uptime, incidents) | Sentry-compatible error tracking plus logs, infra metrics, and incident/on-call management in one platform -- reduces the number of vendors for a product this size (25 features) | Volume-based pricing, no per-seat or per-host fees | Low-Medium -- single platform covers error tracking, logging, and status pages together | Newer consolidated offering; moderate lock-in if adopting its bundled incident-management workflow | https://www.dash0.com/comparisons/best-sentry-alternatives (accessed 2026-09-28) |
| New Relic | Full-stack observability platform | Comparable breadth to Datadog (APM, infra, logs); alternative full-platform option for unified tracing across background jobs and real-time paths | Usage-based (data ingest + user seats); free tier available | Medium -- comparable setup effort to Datadog | Very mature; moderate-high lock-in via its proprietary query language (NRQL) | https://costbench.com/best/best-error-tracking-tools/ (accessed 2026-09-28) |

### File & Object Storage

Serves the single-recipe web import path (no binary uploads: FEAT-10 stores extracted text/structured data, not files) and the whole-household data export (FEAT-18.SPEC-001, FEAT-18.SPEC-006) -- generated export artifacts (JSON/CSV-class files) needing temporary or durable storage and a download link, not user media uploads (File upload = No in profile Section 3).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Cloudflare R2 | Managed object storage | Zero egress cost fits one-off export downloads (FEAT-18.SPEC-013 Export Ready Notification links to a file the household downloads once) | $0.015/GB-month storage, $0 egress, $15/TB for Class A operations | Low -- S3-compatible API, broad SDK support | Mature; low lock-in via S3 API compatibility | https://cloudzat.com/object-storage/ (accessed 2026-09-28) |
| Amazon S3 | Managed object storage | Deepest ecosystem and lifecycle-policy tooling (useful for auto-expiring export files); higher egress cost is a minor factor given low per-household export frequency | $0.023/GB-month storage; egress ~$0.09/GB | Low -- broadest SDK/tooling coverage of the options researched | Extremely mature; low data lock-in (S3 API is the de facto standard others mimic) | https://cloudzat.com/object-storage/ (accessed 2026-09-28) |
| Supabase Storage | Managed object storage (BaaS) | Bundled with Supabase if chosen as the Database/Auth option, avoiding a third vendor relationship for a low-volume export-file use case | Pro plan $25/month incl. 100GB storage, 250GB egress; overage $0.021/GB storage, $0.09/GB egress | Low if Supabase is already adopted elsewhere -- otherwise adds a service with overlapping scope | Mature; moderate lock-in tied to the Supabase platform | https://adamarant.com/en/blog/cloudflare-r2-vs-s3-vs-supabase-storage-in-2026-which-to-pick (accessed 2026-09-28) |
| Backblaze B2 | Managed object storage | Lower-cost alternative for the same use case (small, infrequent export files); S3-compatible API | Storage and egress priced below AWS S3, comparable positioning to R2 | Low -- S3-compatible API | Mature; low lock-in via S3 API compatibility | https://cloudzat.com/object-storage/ (accessed 2026-09-28) |

### Email & Messaging Delivery

Serves 20 Notification specs across push and transactional email channels (ASMP-31 device-notification delivery, ASMP-32 transactional email) -- account recovery, plan-ready alerts, billing notices, safety-concern reports, and support acknowledgements.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Resend | Managed email API | Developer-friendly API and template tooling for transactional email (account recovery, plan-ready fallback, billing notices) | Free tier 3,000 emails/month (permanent); paid tiers above that | Low -- REST API plus official Node SDK | Newer but fast-growing; SMTP fallback limits lock-in | https://ventureharbour.com/transactional-email-service-best-mandrill-vs-sendgrid-vs-mailjet/ (accessed 2026-09-28) |
| Postmark | Managed email API | Fast, reliable transactional-only delivery (sub-2-second average) -- fits time-sensitive notices like payment-failure grace-period alerts | Free tier 100 emails/day; paid from $19.95/month for 50,000 emails; dedicated IPs from $89.95/month | Low -- REST API plus SDKs | Long-established, transactional-first reputation; message-stream model is proprietary but content is exportable | https://www.emailtooltester.com/en/blog/best-transactional-email-service/ (accessed 2026-09-28) |
| SendGrid (Twilio) | Managed email + SMS platform | Bundles email with Twilio's SMS capability, relevant if push notifications need an SMS fallback beyond the stated push/email channels | No free plan as of the researched source; paid from usage-based tiers | Low-Medium -- REST API plus SDKs, larger platform to configure | Very mature, high scale ceiling; coupled to a Twilio account | https://ventureharbour.com/transactional-email-service-best-mandrill-vs-sendgrid-vs-mailjet/ (accessed 2026-09-28) |
| OneSignal | Managed push notification platform | Quickest self-serve push setup for the daily "tonight's dinner" nudge (FEAT-13) and plan-ready push alert (FEAT-07) | Free tier available; paid tiers scale with audience size | Low -- widely documented Web Push and mobile-web SDK integration | Mature, widely adopted for push specifically | https://www.courier.com/blog/top-push-notification-platforms (accessed 2026-09-28) |
| Courier | Unified notification API (multi-channel) | Single API for push, email, and SMS across FCM/APNs/Expo/SendGrid/Twilio -- reduces per-channel vendor wiring for the 20 Notification specs spanning both channels | $0.005/message across channels, 10,000 messages/month free | Low-Medium -- one integration point but an added abstraction layer over the underlying providers | Newer aggregator layer; moderate lock-in to Courier's routing/template model on top of the underlying providers | https://www.courier.com/blog/top-push-notification-platforms (accessed 2026-09-28) |

### Payments & Billing

Serves FEAT-14's full subscription lifecycle (tier overview, upgrade, downgrade/cancel, grace-period handling, refunds, billing-access authorization) plus free-tier default provisioning (FEAT-01.SPEC-011) and the founder's roughly-three-month revenue timeline (ASMP-33).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Stripe Billing | Payment processing + subscription primitives | Payment primitives plus subscription objects the team assembles into the tier/grace-period/refund logic FEAT-14 specifies | 2.9% + $0.30 per charge (base processing); Billing adds 0.7% of billing volume or annual tiers from $620/month | Low-Medium -- extensive docs and webhooks, but the team builds dunning/grace-period logic on top | Very mature, the default payments integration for the frameworks researched above; low-moderate lock-in (well-documented data export) | https://unibee.dev/blog/paddle-vs-stripe-the-ultimate-comparison/ (accessed 2026-09-28) |
| Paddle | Merchant-of-record payment platform | Handles tax/VAT/GST compliance, chargebacks, and revenue recovery as part of the platform -- reduces compliance surface for a two-geography (US/UK) launch | 5% + $0.50 per checkout transaction, all-inclusive | Low -- less custom billing logic to build than Stripe, at a higher per-transaction cost | Mature; higher lock-in as merchant-of-record (Paddle is the seller of record, not the founder's entity) | https://unibee.dev/blog/paddle-vs-stripe-the-ultimate-comparison/ (accessed 2026-09-28) |
| Chargebee (on top of Stripe) | Subscription billing management layer | Provides the tier/dunning/grace-period logic FEAT-14 specifies as configuration rather than custom code, sitting atop a processor like Stripe | Starter plan free up to $250,000 cumulative billing; Performance plan $599/month for revenue up to $100K/month | Medium -- a second platform on top of the payment processor, more configuration surface | Mature; moderate lock-in to Chargebee's billing data model layered over the processor | https://baremetrics.com/blog/stripe-vs-chargebee-features-pricing-reviews-and-more (accessed 2026-09-28) |
| Recurly | Subscription billing management layer | Dunning-focused subscription billing, comparable positioning to Chargebee for grace-period and payment-failure handling (FEAT-14.SPEC-007) | Usage/revenue-based pricing tiers (not itemized in the researched source) | Medium -- similar integration profile to Chargebee | Mature, established subscription-billing specialist | https://unibee.dev/blog/paddle-vs-stripe-the-ultimate-comparison/ (accessed 2026-09-28) |

### AI & Intelligent Behavior

Serves weekly dinner plan generation and swap-alternatives generation (FEAT-03.SPEC-010, FEAT-04.SPEC-007) plus tier-gated learned preference weighting (FEAT-12.SPEC-004) -- BRIEF.md states "the founder has no preference on which one" and the AI cost per household must stay small, roughly one weekly plan plus a few swaps, with the free tier getting no AI.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Anthropic Claude API (e.g., Sonnet-tier model) | Managed LLM API | Strong structured-output and instruction-following for plan/swap generation with an explained-wait UX (ASMP-23); prompt caching helps keep per-household cost small | ~$2/$10 per 1M input/output tokens at the mid tier researched; caching drops cache-hit cost to 0.1x base input | Low -- REST API and SDKs, provider-agnostic prompt design keeps switching cost low | Mature, actively developed; low-moderate lock-in via standard REST/JSON API | https://www.cloudzero.com/blog/llm-api-pricing-comparison/ (accessed 2026-09-28) |
| OpenAI GPT API (mid-tier model) | Managed LLM API | Comparable capability profile to Claude for structured plan/swap generation; Batch API cuts costs 50% for non-interactive batch generation runs | Mid-tier researched at ~$2/$10 per 1M input/output tokens; Batch API halves rates | Low -- REST API and SDKs | Mature, broad ecosystem; low-moderate lock-in via REST API | https://intuitionlabs.ai/articles/llm-api-pricing-comparison-2025 (accessed 2026-09-28) |
| Google Gemini API | Managed LLM API | Comparable pricing tier (Gemini 3.1 Pro at $2/$12 per 1M tokens); fits a provider-agnostic selection per the founder's stated indifference | Gemini 3.1 Pro $2/$12 per 1M tokens; Gemini 3.8 Flash $0.75/$3.75 (promotional through 2026-12-31) | Low -- REST API and SDKs | Mature; low-moderate lock-in via REST API | https://www.spheron.network/blog/llm-api-pricing-comparison-gpt-claude-gemini-deepseek-2026/ (accessed 2026-09-28) |
| Self-hosted open-weight model (e.g., via a GPU-hosting provider) | Self-hosted inference | Full cost/latency control and no per-token vendor markup at scale; requires the team to own model selection, prompt tuning, and inference infrastructure -- disproportionate operational cost for "one weekly plan plus a few swaps" per household (BRIEF.md) | Infrastructure cost (GPU compute) rather than per-token API pricing | High -- team owns hosting, scaling, and model updates | Rapidly evolving open-weight ecosystem; low data lock-in, high operational ownership | knowledge-based -- no single vendor pricing page applies to a self-hosted inference approach; general pattern from model knowledge |

### Search

Serves recipe-library browse-and-search by name or ingredient with dietary-badge filtering (FEAT-08.SPEC-001) -- explicitly a simple-filter complexity, not full-text or faceted search, per profile Section 3.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostgreSQL full-text search | Native database feature | Free and sufficient for small-to-medium recipe catalogs and simple name/ingredient/badge filtering -- avoids a dedicated search service for this product's stated simple-filter complexity | Included with the Database area's Postgres options; no additional cost | Low -- no new service, works directly against the chosen database | Very mature; zero additional lock-in (native to the database already selected) | https://layerbase.com/blog/algolia-alternatives (accessed 2026-09-28) |
| Meilisearch | Search engine (open-source, managed cloud option) | Typo-tolerant, fast search if the recipe library grows past what Postgres full-text search comfortably serves; per-instance pricing avoids per-query cost surprises | Open-source self-hosted free; Meilisearch Cloud from roughly the low hundreds of dollars/month at moderate scale | Medium -- a new service and index-sync pipeline versus native database search | Mature open-source project; low lock-in (self-hostable, open data format) | https://layerbase.com/blog/algolia-vs-meilisearch (accessed 2026-09-28) |
| Typesense | Search engine (open-source, managed cloud option) | Comparable positioning to Meilisearch -- "80% of features at 20% of cost" versus Algolia, open-source and self-hostable | Self-hosted free; Typesense Cloud from ~$30/month | Medium -- new service and index-sync pipeline | Mature open-source project; low lock-in | https://tech-insider.org/typesense-vs-algolia-vs-meilisearch-2026/ (accessed 2026-09-28) |
| Algolia | Managed search platform (SaaS) | Highest-feature, highest-cost option; disproportionate for a simple-filter recipe search at this product's scale (several thousand households, one starter recipe library) | Free to 10,000 searches/1M records/month; Build plan $0.50/1,000 searches; Grow from $550/month at 100K searches | Low -- managed indexing and query API, minimal ops | Very mature; moderate lock-in to Algolia's indexing/query model and cost structure at scale | https://layerbase.com/blog/algolia-vs-meilisearch (accessed 2026-09-28) |

### Background Jobs & Scheduling

Serves 7+ scheduled/background automations (FEAT-03.SPEC-003 weekly plan generation, FEAT-07.SPEC-001 plan-ready trigger, FEAT-13.SPEC-001 daily nudge, FEAT-06.SPEC-004 week rollover, FEAT-25.SPEC-003 weekly check-in, FEAT-09.SPEC-006 invitation expiry, FEAT-04.SPEC-005 suggestion lapse) plus export generation (FEAT-18.SPEC-006) and notification delivery.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Inngest | Managed durable-workflow platform | Step-based durability with retries fits scheduled weekly generation, daily nudges, and multi-step export processing without a separate worker deployment | Free tier 25,000 runs/month; paid from $75/month | Low -- zero-infra setup, event-driven functions written in the backend language | Mature, widely recommended default for early-stage Next.js/Node SaaS in the researched source | https://upstash.com/blog/serverless-background-jobs-and-message-queues-every-major-option-in-2026 (accessed 2026-09-28) |
| Trigger.dev | Managed durable-workflow platform (open-source core) | Comparable durable-execution model to Inngest, open-source with a self-host option if data-residency concerns arise from the compliance posture (ASMP-27) | Free tier 50,000 runs/month | Low -- plain TypeScript task definitions, durable execution, no function timeouts | Mature; lower lock-in than fully-managed-only competitors via the self-host option | https://starterpick.com/guides/background-jobs-inngest-vs-bullmq-vs-trigger-2026 (accessed 2026-09-28) |
| Upstash QStash | Managed HTTP-based message queue | Simple "deliver an HTTP request on schedule/queue" model -- lightweight fit for the plan-ready and nudge triggers without adopting a full workflow-orchestration platform | 500 messages/day free; usage-based beyond that | Low -- no worker/consumer to deploy, just an HTTP endpoint | Mature (Upstash); low lock-in via plain HTTP delivery | https://upstash.com/blog/serverless-background-jobs-and-message-queues-every-major-option-in-2026 (accessed 2026-09-28) |
| BullMQ (self-hosted, Redis-backed) | Open-source job queue library | Full control over retry/backoff semantics with no per-run pricing -- cost stays flat as job volume grows across the 7+ scheduled automations | MIT-licensed, free; cost is Redis hosting ($5-10/month on Upstash/Redis Cloud for small workloads) plus worker compute | Medium -- team deploys and scales its own worker processes against Redis | Very mature, battle-tested Node.js queue library; low lock-in (open source, standard Redis) | https://upstash.com/blog/serverless-background-jobs-and-message-queues-every-major-option-in-2026 (accessed 2026-09-28) |

### Caching & Performance

Serves the "instant" grocery-list tick/add/edit target and few-seconds swap propagation (ASMP-22), the under-a-minute plan-generation and ~1-second recipe-search targets (ASMP-23), and offline resilience in low-signal shopping conditions (FEAT-06.SPEC-008).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Upstash Redis | Managed serverless cache | Pay-per-request model fits variable household traffic; HTTP API works from edge/serverless functions for low-latency cache reads on the grocery-list hot path | $0.20 per 100,000 commands, first 500,000/month free; storage $0.25/GB | Low -- HTTP-based client works in edge and serverless runtimes without a persistent connection | Mature; low lock-in (Redis-API-compatible) | https://upstash.com/blog/redis-pricing-comparison-every-major-provider-in-2026-with-numbers (accessed 2026-09-28) |
| Redis Cloud | Managed traditional cache | Better raw throughput at high, steady command volume; becomes cheaper than pay-per-request Upstash at roughly 5-10M commands/month | From ~$5/month (250MB); $70/month minimum typically cited for production tiers | Low-Medium -- standard Redis connection, no HTTP-adapter layer needed | Mature, the managed version of standard Redis; low lock-in (standard Redis protocol) | https://upstash.com/blog/redis-pricing-comparison-every-major-provider-in-2026-with-numbers (accessed 2026-09-28) |
| Service-worker / IndexedDB client-side caching (e.g., via a PWA layer) | Client-side caching library/pattern | Directly serves the offline-queuing requirement (FEAT-01.SPEC-013, FEAT-06.SPEC-008) by persisting drafts and queued grocery-list edits on-device for later sync | No vendor cost; implementation effort only | Medium -- requires explicit service-worker and sync-queue implementation in the frontend | Mature browser-standard APIs; zero vendor lock-in | knowledge-based -- this is a browser-platform capability rather than a single vendor's product, so no single pricing/comparison URL applies |
| Vercel KV (Upstash-backed) | Managed serverless cache (platform-integrated) | Same Upstash Redis engine with tighter Vercel platform integration if Vercel is the Hosting & Environments choice | 2x per-command cost versus using Upstash directly | Low if already on Vercel -- adds convenience at a cost premium | Mature underlying engine; higher lock-in (Vercel-platform-specific wrapper over Upstash) | https://upstash.com/blog/upstash-redis-vs-cloudflare-kv (accessed 2026-09-28) |

### Real-time & Collaboration

Serves live plan and grocery-list propagation across household devices (FEAT-03.SPEC-011, FEAT-06.SPEC-005) with extensive last-write-wins/reject-with-refresh concurrency rules across 16 of 17 entities (profile Section 3, Collaboration/concurrency) and offline reconciliation (FEAT-06.SPEC-008).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Supabase Realtime | Managed realtime (Postgres-native) | Built directly on Postgres change-data-capture -- if Supabase is the Database choice, live plan/grocery-list updates require no separate realtime service | Included in Supabase Pro ($25/month) for typical household-scale concurrent connections (~10,000 MAU/200 concurrent) | Low if Supabase is already the database -- otherwise a mismatch requiring a second Postgres-adjacent service | Mature; lock-in tied to the Supabase platform | https://www.agilesoftlabs.com/blog/2026/05/supabase-realtime-in-production-what (accessed 2026-09-28) |
| Ably | Managed realtime pub/sub platform | Guaranteed ordering, exactly-once delivery, and connection recovery out of the box -- fits the reject-with-refresh and last-write-wins concurrency rules that depend on reliable delivery order | Connection-hours based; ~$30/month for 200 concurrent connections across a month | Low-Medium -- SDKs for the frontend frameworks researched, independent of the database choice | Mature, enterprise-grade realtime infrastructure; moderate lock-in to Ably's channel/presence model | https://ably.com/compare/ably-vs-supabase (accessed 2026-09-28) |
| Pusher (Channels) | Managed realtime pub/sub platform | Simple, predictable-pricing WebSocket channels with presence -- lighter-weight than Ably for the household-scoped (2-6 members) channel model this product needs | 200,000 messages/day and 200 concurrent connections free | Low -- widely documented, simple channel-subscribe model | Mature, the most widely adopted managed pub/sub service per the researched source; moderate lock-in | https://ably.com/compare/pusher-vs-supabase (accessed 2026-09-28) |
| Socket.io (self-hosted) | Open-source realtime library | Full control over the sync/conflict-resolution logic the product's 16-of-17-entity contention rules require; scales horizontally via Redis adapter | Open source; free (infrastructure cost for hosting + Redis for scaling) | High -- team owns connection scaling, presence, and reconnection logic | Very mature, battle-tested; zero vendor lock-in (self-hosted, open protocol) | https://www.pkgpulse.com/guides/best-realtime-libraries-2026 (accessed 2026-09-28) |

### Analytics & Product Telemetry

Serves the founder's stated need to track several thousand households in year one while "staying equally responsive as that base grows" (ASMP-24) -- event-level product analytics to observe usage and performance trends over time.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostHog | Product analytics platform (open-source, self-hostable) | Bundles analytics, session replay, and feature flags in one developer-controlled stack -- useful for an engineering-led team tracking a several-thousand-household base | 1M events/month free; ~$0.00031/event beyond that (roughly $1,240/month at 5M events) | Low -- SDKs for the frontend/backend frameworks researched, self-host option available | Mature, fast-growing open-source option; low lock-in via self-host, moderate on managed cloud | https://fastero.com/blog/posthog-vs-amplitude-vs-mixpanel-product-analytics-showdown (accessed 2026-09-28) |
| Mixpanel | Product analytics platform | Simplest event-based analytics setup with a generous free tier, fast funnel/retention analysis for household-engagement metrics | Free tier up to 20M events/month | Low -- straightforward event-tracking SDKs | Mature; moderate lock-in to Mixpanel's event schema and query model | https://fastero.com/blog/posthog-vs-amplitude-vs-mixpanel-product-analytics-showdown (accessed 2026-09-28) |
| Amplitude | Product analytics platform | Mature behavioral analytics with AI-powered insights, suited to tracking cohort responsiveness as the household base grows (ASMP-24) | Free Starter under 50,000 MTU; Growth from ~$49/month, scaling with tracked-user count | Low-Medium -- SDK integration comparable to Mixpanel | Mature; moderate lock-in to Amplitude's behavioral-cohort model | https://fastero.com/blog/posthog-vs-amplitude-vs-mixpanel-product-analytics-showdown (accessed 2026-09-28) |

### Internationalization

Serves configurable units (cups/oz vs grams/ml), currency (at least USD and GBP), and supermarket aisle names for US and UK households from launch, English-only (FEAT-16.SPEC-001, FEAT-16.SPEC-002, ASMP-28) -- a locale-data/formatting need more than a multi-language translation need.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| next-intl | i18n library (Next.js-native) | Server Component-native locale/number/currency formatting if Next.js is the frontend choice; proper per-locale handling fits the US/GBP unit-and-currency configuration need | Open source; free | Low -- purpose-built for Next.js App Router, minimal setup for the two-locale (US/UK) scope this product needs | Mature, ~900K weekly downloads; low lock-in (standard ICU-based formatting) | https://tolgee.io/blog/react-i18n-libraries-comparison (accessed 2026-09-28) |
| react-i18next | i18n library (framework-agnostic) | Works across every frontend framework researched (React, Next.js, Remix, Vite-based apps); largest ecosystem if the frontend choice isn't Next.js specifically | Open source; free | Low-Medium -- most widely used, but pairing with ICU is recommended for platform-compatible formatting | Very mature, largest i18n ecosystem for React; low lock-in | https://dev.to/erayg/best-i18n-libraries-for-nextjs-react-react-native-in-2026-honest-comparison-3m8f (accessed 2026-09-28) |
| FormatJS (react-intl) | i18n library | Strong ICU MessageFormat support for rigorous number/date/plural/currency formatting -- directly serves the units-and-currency configuration requirement (FEAT-16) | Open source; free | Low-Medium -- comparable integration effort to react-i18next | Mature; low lock-in, standards-based (ICU) | https://phrase.com/blog/posts/react-i18n-best-libraries/ (accessed 2026-09-28) |
| LinguiJS | i18n library | Tiny runtime (~5KB gzipped); source-code string extraction suits a small, fixed locale set (US/UK, English-only per ASMP-28) without a large translation-management workflow | Open source; free | Low-Medium -- requires Babel/SWC macro tooling for extraction | Mature; low lock-in, smallest bundle footprint of the options researched | https://www.pkgpulse.com/guides/next-intl-vs-react-i18next-vs-lingui-react-i18n-2026 (accessed 2026-09-28) |

### Recipe & Food Content Data

Serves seeding the starter recipe library at launch with complete ingredient data the allergy-safety engine (FEAT-02) can verify (ASMP-34, FEAT-08.SPEC-004 Starter Recipe Content Seeding & Maintenance) -- a licensed recipe/ingredient dataset, not a search-index technology.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Spoonacular Food API | Managed content data API | ~365,000-recipe database with recipe analysis and ingredient/nutrition data suited to seeding a starter library the allergy engine can parse | From $300/month at the tier researched (another source cites a range from free to $149/month depending on plan) | Low-Medium -- REST API, requires a one-time or periodic seeding pipeline plus ongoing maintenance sync (FEAT-08.SPEC-004) | Mature, established recipe-data vendor; moderate lock-in to Spoonacular's ingredient taxonomy for allergen matching | https://blog.nutrigraphapi.com/best-food-apis-for-developers-in-2026-edamam-vs-spoonacular-vs-nutritionix-vs-nutrigraphapi/ (accessed 2026-09-28) |
| Edamam Recipe Search API | Managed content data API | Largest publicly indexed recipe count (2M+ aggregated) plus a smaller owned-content set (20,000+ recipes with images/instructions) and strong nutrition-analysis specialization for allergen/dietary-rule matching | Nutrition analysis API from $299/month at the researched tier (another source cites a range up to $999/month) | Low-Medium -- REST API, similar seeding-pipeline effort to Spoonacular | Mature, specializes in nutritional compliance data; moderate lock-in to Edamam's data schema | https://developer.edamam.com/edamam-recipe-api (accessed 2026-09-28) |
| Curated/licensed starter set + manual editorial seeding | Manual/self-sourced content | Full control over data completeness and allergen-field accuracy for the safety engine, at the cost of a manual content-production effort to reach a usable starter-library size | One-time content-licensing/production cost rather than a recurring API fee | High -- team (or a content vendor) authors and maintains the ingredient-complete starter library directly | No vendor lock-in; highest control over the allergen-completeness fail-closed policy (FEAT-02.SPEC-007) | knowledge-based -- this is a content-production approach rather than a single vendor's product, so no comparison URL applies |
| Tasty/Yummly-class licensed content partnerships | Licensed media/content partnership | Larger media-company recipe catalogs sometimes available via content-licensing deals rather than a public self-serve API; would need direct vendor outreach to confirm current terms | Not publicly published; typically negotiated licensing terms | Medium-High -- requires a licensing negotiation rather than a self-serve API signup | Mature media brands; lock-in and terms depend entirely on the negotiated license | knowledge-based -- these programs are not consistently self-serve/publicly priced, and the search results returned did not surface a current public pricing page for this category |

### Web Page Recipe Extraction

Serves reading ingredients and steps from a pasted recipe link (FEAT-10.SPEC-001 Import by Link, FEAT-10.SPEC-005 Web Page Recipe Extraction) -- structured-data extraction from arbitrary recipe websites, noted in the profile as legally unresolved for v1 (ASMP-36, BRIEF.md Open Questions).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| recipe-scrapers (open-source library) | Self-hosted library | Parses schema.org/Recipe JSON-LD, Microdata, and OpenGraph metadata directly -- covers the majority of modern recipe sites without a third-party service dependency | Open source; free | Medium -- self-hosted parsing logic the team maintains as source sites change markup | Mature open-source project; zero vendor lock-in | https://docs.recipe-scrapers.com/ (accessed 2026-09-28) |
| Custom JSON-LD/schema.org parser (self-built) | Self-built extraction logic | Directly targets the same schema.org/Recipe markup virtually every major cooking site publishes -- reported as reliable on 95%+ of recipe sites via JSON-LD alone | Build/maintenance engineering cost only | Medium -- team builds and maintains the parser against evolving site markup | No vendor dependency; full control but full maintenance burden | knowledge-based -- confirms the JSON-LD/schema.org approach described in the researched Apify/recipe-scrapers sources; this specific "self-built" framing summarizes that pattern rather than citing a single vendor page |
| Managed recipe-extraction API (e.g., Apify recipe-scraper actors) | Managed scraping-as-a-service | Third-party-hosted extraction across 100-725+ recipe sites depending on the specific actor, reducing in-house maintenance of site-specific parsing | Usage-based, per-run/per-page pricing on the hosting marketplace (Apify) | Low -- API call replaces self-hosted parsing/maintenance | Mature marketplace, individual actors vary in maintenance quality; moderate lock-in to the marketplace platform | https://apify.com/crawlerbros/recipe-scraper/api (accessed 2026-09-28) |
| AI-fallback extraction (JSON-LD/Microdata/heuristic parsing + LLM fallback) | Hybrid self-built + managed LLM API | Four-tier approach -- structured data first, LLM fallback only for unstructured pages -- keeps per-import AI cost low while covering sites without schema.org markup, aligning with the profile's small-AI-cost expectation (BRIEF.md) | Structured-data tiers free; LLM fallback billed per the AI & Intelligent Behavior area's token pricing, only invoked on parse failures | Medium -- combines a self-built parser with a fallback call to the AI & Intelligent Behavior area's chosen LLM provider | Demonstrated pattern (e.g., RecipeStripper's four-tier parser); lock-in is limited to whichever LLM provider is chosen for the fallback tier | https://recipestripper.com/recipe-extractor (accessed 2026-09-28) |

### Online Grocery Ordering Integration

Serves the Later-phase Online Grocery Ordering Handoff (FEAT-20, FEAT-20.SPEC-002) -- sending the grocery list out and receiving acceptance/rejection and item-availability responses back, explicitly named with Instacart and Tesco as examples (BRIEF.md, Ecosystem & Integrations).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Instacart Connect / Instacart Developer Platform (IDP) | Managed partner API | Explicitly named in BRIEF.md; self-service developer platform (as of July 2026) with fulfillment, item-availability, and replacement-item lookup APIs matching FEAT-20.SPEC-002's payload/response shape, for the US market | Partner/commercial terms via the developer dashboard; not a flat public price in the researched source | Medium -- API-key-based self-service onboarding, but full fulfillment integration has real-world logistics scope | Mature, actively expanding self-service platform; moderate lock-in to Instacart's API/catalog model, US-market-only | https://docs.instacart.com/connect (accessed 2026-09-28) |
| Grocery aggregator APIs (e.g., listed on API marketplaces) | Managed aggregator API | Third-party marketplaces list multiple grocery-retailer APIs (including UK-relevant options) under one discovery point, useful given no dedicated Tesco developer API was found in this run's research | Varies by individual API listed; typically usage-based | Medium -- requires evaluating each listed retailer API individually for UK/Tesco-equivalent coverage | Varies by underlying provider; aggregator itself is a discovery layer, not a single stable vendor relationship | https://rapidapi.com/collection/grocery (accessed 2026-09-28) |
| Direct retailer partnership/custom integration (e.g., a UK grocer such as Tesco) | Custom partner integration | BRIEF.md names Tesco as a desired example; no public self-service developer API for Tesco surfaced in this run's research, so UK coverage would require direct commercial/API partnership outreach | Not publicly published; negotiated per retailer | High -- no self-serve path found; requires business development plus custom integration work | Unknown maturity per retailer; lock-in and terms depend entirely on the negotiated partnership | knowledge-based -- the search results returned confirmed Instacart's public API but did not surface a public, self-service Tesco (or other UK grocer) developer API; flagged here as a research gap rather than a fabricated vendor claim |

### Family Calendar Integration

Serves the Later-phase Family Calendar Sync (FEAT-21, FEAT-21.SPEC-002) -- connecting/disconnecting a calendar, creating/updating a per-night dinner entry, and receiving inbound sync-outcome and disconnect events (BRIEF.md, Ecosystem & Integrations: "a nice-to-have for showing dinner on the family calendar").

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Google Calendar API (direct) | Managed provider API (single-provider) | Free and sufficient if the household's calendar is Google Workspace/Gmail-based, which is plausible for a consumer household product | Free | Medium -- team builds and maintains the OAuth flow, event CRUD, and sync-outcome handling directly against one provider | Mature, well-documented; low-moderate lock-in (standard Google OAuth/API), but becomes a liability once Microsoft 365/Apple support is needed | https://www.cronofy.com/blog/best-calendar-apis (accessed 2026-09-28) |
| Cronofy | Unified calendar API (managed) | Real-time availability and calendar-write API across Google, Microsoft, and Apple/Exchange under one interface -- covers the range of consumer calendar providers a household might use, without Nylas's bundled email/contacts scope | Usage-based with a $99/month minimum for standalone use; plan-based Starter tier from $139/month | Low -- one API for multi-provider coverage instead of building per-provider OAuth flows | Mature, calendar/scheduling specialist; moderate lock-in to Cronofy's unified event model | https://www.cronofy.com/blog/best-calendar-apis (accessed 2026-09-28) |
| Nylas | Unified calendar + email API (managed) | Broader bundled scope (calendar, email, contacts) than this feature needs (only per-night dinner entries and connect/disconnect are specified); per-connected-account pricing model | Per-connected-account pricing (not itemized in the researched source) | Low-Medium -- one API for multi-provider calendar coverage, but adopts a broader platform than the feature's scope requires | Mature; the researched source notes a forced migration prompting some teams to seek alternatives -- a lock-in signal worth weighing | https://www.nylas.com/blog/best-calendar-apis/ (accessed 2026-09-28) |
| Direct multi-provider integration (Google Calendar API + Microsoft Graph API, self-built) | Self-built multi-provider integration | Full control over exactly the connect/disconnect and per-night-entry sync FEAT-21 specifies, without adopting a third-party unified-API vendor's broader data model | Both provider APIs are free to use; cost is engineering time | High -- team builds and maintains two separate OAuth/event-sync integrations instead of one unified API | Both underlying provider APIs are mature; zero third-party API vendor lock-in, but higher maintenance burden as each provider's API evolves | https://www.cronofy.com/blog/best-calendar-apis (accessed 2026-09-28) |

## 3. Cross-Area Compatibility Notes

- Prisma and Drizzle (ORM / Data Access) are TypeScript/Node data-access layers -- they pair with the Node-runtime Backend / API Layer candidates (NestJS, Fastify, Express, Next.js Route Handlers) researched above, not with the Django (Python) candidate. SQLAlchemy pairs only with a Python backend such as Django; it does not pair with any Node-runtime Backend / API Layer candidate researched here.
- Every managed-Postgres Database candidate researched (Neon, Supabase, Amazon RDS, PlanetScale for Postgres, self-hosted Postgres) is wire-compatible with the standard Postgres protocol -- all four ORM / Data Access candidates that support Postgres (Prisma, Drizzle, TypeORM) work against any of them without a dialect change.
- Pinia (State Management) is Vue-only and pairs only with the Nuxt 3 Frontend Framework candidate. SWR-style server-cache libraries and Redux Toolkit/Jotai (State Management) are React-focused and pair with the Next.js and Remix Frontend Framework candidates, not Nuxt or SvelteKit.
- The component-layer candidates named under CSS / Styling follow the same binding facts: shadcn/ui and Radix Primitives assume a React frontend (Next.js or Remix from the Frontend Framework options); Headless UI supports both React and Vue frontends (Next.js/Remix or Nuxt); Melt UI is Svelte-only and pairs only with the SvelteKit Frontend Framework candidate.
- Turbopack (Build Tooling) is Next.js-specific and is only a meaningful choice if Next.js is the Frontend Framework selection; Vite is the build tool for the Nuxt, SvelteKit, and Remix Frontend Framework candidates researched.
- Supabase appears as a candidate in four separate areas (Database, Authentication & Identity, File & Object Storage, Real-time & Collaboration) with bundled pricing -- selecting Supabase as the Database consolidates Auth, Storage, and Realtime under one Pro-tier bill ($25/month base) rather than requiring four separate vendor relationships; each of those four areas' non-Supabase candidates would instead require independent integration and billing.
- Clerk and Supabase Auth (Authentication & Identity) both ship official Next.js/React SDKs matching the Next.js and Remix Frontend Framework candidates; Better Auth and NextAuth.js are self-hosted and framework-agnostic at the cost of the team building the role/session model FEAT-01.SPEC-016's authorization rules require.
- Resend, Postmark, and SendGrid (Email & Messaging Delivery) all ship first-party Node SDKs matching every Backend / API Layer candidate researched; Courier's multi-channel API wraps several of the same underlying providers (including SendGrid and Twilio) rather than replacing them.
- Vercel KV (Caching & Performance) is the same Upstash Redis engine as the standalone Upstash Redis candidate, wrapped with Vercel-platform billing at roughly 2x the per-command cost -- it is only a distinct choice from plain Upstash Redis when Vercel is also the Hosting & Environments selection.
- Inngest, Trigger.dev, and Upstash QStash (Background Jobs & Scheduling) all integrate via plain HTTP endpoints or SDK calls from any of the Backend / API Layer candidates researched; BullMQ additionally requires a Redis instance, which can be satisfied by either of the Caching & Performance area's Upstash Redis or Redis Cloud candidates.
- The Web Page Recipe Extraction area's AI-fallback option depends on whichever provider is selected under AI & Intelligent Behavior (Anthropic Claude, OpenAI GPT, or Google Gemini) for its fallback tier -- the two areas are not independent once that hybrid approach is chosen.
- Cronofy and Nylas (Family Calendar Integration) both cover Google, Microsoft, and Apple/Exchange calendars under one API, so either is a superset of the single-provider Google Calendar API (direct) candidate researched in the same area; neither the Online Grocery Ordering Integration nor Family Calendar Integration candidates researched bind to a specific Frontend Framework, Backend / API Layer, or Database choice -- both are server-side HTTP integrations callable from any Backend / API Layer candidate researched.

## 4. Research Log

| Decision Area | Method | Queries & Key Sources | Access Date |
|---|---|---|---|
| Frontend Framework | web | "best frontend frameworks 2026 React Next.js Vue Nuxt SvelteKit comparison for mobile-first web app"; intuz.com, brilworks.com, mgsoftware.nl, coderio.com, midrocket.com | 2026-09-28 |
| Backend / API Layer | web | "Node.js backend framework 2026 NestJS Express Fastify comparison"; index.dev, dev.to (ihor_ostin), tech-insider.org, quartzdevs.com; dev.to (nayankyada) for Next.js Route Handlers context | 2026-09-28 |
| Database | web (one knowledge-based row) | "managed PostgreSQL hosting pricing 2026 Neon Supabase PlanetScale RDS comparison"; bytebase.com, dev.to (philip_mcclarence_2ef9475), dreamlit.ai, selfhost.dev -- self-hosted Postgres row is a general operational pattern with no single vendor pricing page, marked knowledge-based | 2026-09-28 |
| ORM / Data Access | web | "Prisma vs Drizzle ORM 2026 comparison TypeScript"; makerkit.dev, bytebase.com, encore.dev (two articles); quartzdevs.com for the SQLAlchemy/Python-backend cross-reference | 2026-09-28 |
| CSS / Styling | web | "Tailwind CSS vs CSS-in-JS vs vanilla-extract 2026 styling comparison"; vibetown.pro, pkgpulse.com, kanopylabs.com, kunalganglani.com | 2026-09-28 |
| State Management | web | "React state management 2026 Zustand Redux Toolkit TanStack Query comparison"; dev.to (iamsaadmehmood), thamizhelango.medium.com, xcodx.io, theroadtoenterprise.com | 2026-09-28 |
| Build Tooling | web | "Vite vs Turbopack vs webpack 2026 build tool comparison"; reintech.io, techsy.io, kunalganglani.com, eneskaymaz.com | 2026-09-28 |
| Authentication & Identity | web | "authentication provider 2026 Clerk Auth0 Supabase Auth NextAuth pricing comparison"; iloveblogs.blog, buildmvpfast.com (api-costs/authentication and blog) | 2026-09-28 |
| Hosting & Environments | web | "hosting platform 2026 Vercel Netlify Render Fly.io pricing comparison web app"; gautamkhorana.com, dev.to (nayankyada), f3fundit.com; dreamlit.ai reused for the AWS RDS cost figure | 2026-09-28 |
| CI/CD & Delivery | web | "CI/CD pipeline 2026 GitHub Actions GitLab CI CircleCI comparison"; devopsboys.com, kunalganglani.com, labhub.hopto.org | 2026-09-28 |
| Observability & Operations | web | "observability error monitoring 2026 Sentry Datadog Better Stack pricing comparison"; betterstack.com, dash0.com, costbench.com | 2026-09-28 |
| File & Object Storage | web | "object storage 2026 AWS S3 Cloudflare R2 Supabase Storage pricing comparison"; cloudzat.com, adamarant.com | 2026-09-28 |
| Email & Messaging Delivery | web | "transactional email push notification service 2026 Resend Postmark SendGrid OneSignal Expo push pricing"; ventureharbour.com, emailtooltester.com, courier.com (two articles) | 2026-09-28 |
| Payments & Billing | web | "payment processing subscription billing 2026 Stripe Paddle Chargebee pricing comparison"; unibee.dev, baremetrics.com | 2026-09-28 |
| AI & Intelligent Behavior | web (one knowledge-based row) | "LLM API pricing 2026 OpenAI GPT Anthropic Claude Google Gemini comparison for text generation"; cloudzero.com, intuitionlabs.ai, spheron.network -- self-hosted open-weight model row is a general pattern with no single vendor pricing page, marked knowledge-based | 2026-09-28 |
| Search | web | "search service 2026 Algolia Typesense Meilisearch Postgres full-text search comparison pricing"; layerbase.com (two articles), tech-insider.org | 2026-09-28 |
| Background Jobs & Scheduling | web | "background job queue scheduler 2026 BullMQ Inngest Trigger.dev QStash comparison pricing"; upstash.com, starterpick.com | 2026-09-28 |
| Caching & Performance | web (one knowledge-based row) | "caching Redis 2026 Upstash Redis Cloud Vercel KV pricing comparison"; upstash.com (two articles) -- service-worker/IndexedDB client-caching row is a browser-platform pattern with no single vendor pricing page, marked knowledge-based | 2026-09-28 |
| Real-time & Collaboration | web | "realtime sync service 2026 Supabase Realtime Pusher Ably PartyKit comparison pricing"; ably.com (two articles), agilesoftlabs.com, pkgpulse.com | 2026-09-28 |
| Analytics & Product Telemetry | web | "product analytics 2026 PostHog Mixpanel Amplitude pricing comparison"; fastero.com | 2026-09-28 |
| Internationalization | web | "i18n internationalization library 2026 next-intl react-i18next Lingui Format.js comparison"; tolgee.io, dev.to (erayg), phrase.com, pkgpulse.com | 2026-09-28 |
| Recipe & Food Content Data | web (two knowledge-based rows) | "recipe database API 2026 Spoonacular Edamam recipe content data provider pricing"; blog.nutrigraphapi.com, developer.edamam.com -- curated/licensed manual-seeding and Tasty/Yummly-class licensed-partnership rows have no self-serve public pricing page, marked knowledge-based | 2026-09-28 |
| Web Page Recipe Extraction | web (one knowledge-based row) | "web page recipe extraction API 2026 recipe scraper schema.org structured data extraction service"; docs.recipe-scrapers.com, apify.com, recipestripper.com -- the "custom JSON-LD/schema.org parser (self-built)" row restates the same researched pattern under a self-built framing with no distinct vendor URL, marked knowledge-based | 2026-09-28 |
| Online Grocery Ordering Integration | web (one knowledge-based row) | "online grocery ordering API integration 2026 Instacart Connect Tesco API developer partnership"; docs.instacart.com, rapidapi.com -- no public self-service Tesco (or other UK grocer) developer API surfaced in this run's search results, so the direct-retailer-partnership row is marked knowledge-based as a documented research gap | 2026-09-28 |
| Family Calendar Integration | web | "calendar sync API 2026 Google Calendar API Apple EventKit Nylas Cronofy comparison"; cronofy.com, nylas.com | 2026-09-28 |


## Feasibility Assessment


# Technical Feasibility Assessment

## 1. Feasibility Summary

| Feature | Verdict | Driving Factors |
|---------|---------|-----------------|
| FEAT-01 (Household Setup & Member Profiles) | Standard-with-integration | Transactional email for sign-up/recovery is an external contract (FEAT-01.SPEC-017 ## Degradation Behavior); offline draft queuing (FEAT-01.SPEC-013 ## Business Rules) and role rules (FEAT-01.SPEC-016) are established patterns |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Hard | Fail-closed, exhaustive allergen matching on every path onto the plan, including compound-term expansion, inside the swap/generation latency budgets (FEAT-02.SPEC-002 ## Processing Logic, ## Edge Cases; FEAT-02.SPEC-007 ## Field Validation Rules; feature-overview.md ## Non-Functional Notes) |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Research-spike recommended | Whether an LLM selecting from the household's candidate pool reliably returns a full 7-night, safety-passing, in-budget week in well under a minute at small per-household cost is unproven (FEAT-03.SPEC-010 ## Data Exchanged, ## Edge Cases; FEAT-03.SPEC-006; ASMP-23, SC-16) |
| FEAT-04 (One-Tap Meal Swap) | Standard-with-integration | AI alternatives are an external call with a specified degradation contract (FEAT-04.SPEC-007 ## Degradation Behavior); the per-slot lock (FEAT-04.SPEC-009 ## Business Rules) is a conventional row-level pattern |
| FEAT-05 (Pantry-Aware Suggestions) | Straightforward | Short household list with exact-name matching and merge-on-duplicate (FEAT-05.SPEC-003, FEAT-05.SPEC-006 ## Business Rules); offline adds reuse the shared offline-queue machinery |
| FEAT-06 (Shared Grocery List) | Hard | Live multi-device sync within 2 seconds plus offline queue reconciliation with idempotent ticks, last-write-wins quantities, deletion precedence and concurrent recalculation merge (FEAT-06.SPEC-005 ## Degradation Behavior; FEAT-06.SPEC-008 ## Cross-Field Rules, ## Edge Cases) |
| FEAT-07 (Weekly Plan Ready Notification) | Standard-with-integration | Push plus email fallback via external delivery services (FEAT-07.SPEC-005, FEAT-07.SPEC-006 ## Degradation Behavior) against an 80%-within-one-minute target (feature-overview.md ## Non-Functional Notes) |
| FEAT-08 (Recipe Library (Starter Recipes)) | Standard-with-integration | Starter corpus depends on an external recipe/food-content source meeting the ingredient-completeness bar (FEAT-08.SPEC-004 ## Data Exchanged, ## Edge Cases); search itself is simple filter (FEAT-08.SPEC-001) |
| FEAT-09 (Household Invitations & Membership) | Straightforward | Link-based invitations, 14-day scheduled expiry and first-write-wins acceptance races are conventional (FEAT-09.SPEC-006; FEAT-09.SPEC-007 ## Edge Cases); notifications are in-app only (FEAT-09.SPEC-012..014 ## Channels) |
| FEAT-10 (Recipe Import from Web Link) | Standard-with-integration | Web-page extraction is an external/self-hosted capability with a defined failure path to manual entry (FEAT-10.SPEC-005 ## Degradation Behavior); legal acceptability is an open product question (ASMP-36) |
| FEAT-11 (Leftover Rollover to Lunches) | Straightforward | Deterministic post-generation slot computation with a two-day ceiling and withdrawal on source change (FEAT-11.SPEC-002 ## Processing Logic; FEAT-11.SPEC-004) |
| FEAT-12 (Meal Rating & Preference Learning) | Straightforward | Ratings are simple records; "learning" is a counted threshold and aggregated weighting input handed to FEAT-03 (FEAT-12.SPEC-004 ## Business Rules; FEAT-12.SPEC-005 ## Processing Logic) |
| FEAT-13 (Tonight's Dinner Reminder) | Standard-with-integration | Daily per-household-local-time push dispatch through the shared device-notification integration, plus a same-day correction (FEAT-13.SPEC-001 ## Trigger Definition; FEAT-13.SPEC-002, FEAT-13.SPEC-004 ## Channels) |
| FEAT-14 (Subscription & Billing Management) | Standard-with-integration | Payment processing with inbound renewal/retry events and a 7-day grace window (FEAT-14.SPEC-009 ## Inbound Events, ## Degradation Behavior; FEAT-14.SPEC-007) |
| FEAT-15 (Member Onboarding) | Straightforward | One-time routing after acceptance, reusing existing plan/list surfaces (FEAT-15.SPEC-002 ## Edge Cases; FEAT-15.SPEC-003) |
| FEAT-16 (Units, Currency & Locale Configuration) | Straightforward | Fixed conversion factors applied at display time, no exchange-rate service (FEAT-16.SPEC-004 ## Cross-Field Rules, ## Scope and Non-Goals) |
| FEAT-17 (Older-Kid Dinner Voting) | Straightforward | Small voting rounds with deterministic resolution (FEAT-17.SPEC-005 ## Processing Logic); depends on an older-kid limited-login role (FEAT-17.SPEC-006 ## Field Validation Rules) |
| FEAT-18 (Account & Data Management) | Standard-with-integration | Export file generation and storage, transactional email, and a 30-day cascade deletion that must reach external services (FEAT-18.SPEC-006, FEAT-18.SPEC-008 ## Processing Logic; FEAT-18.SPEC-012 ## Degradation Behavior) |
| FEAT-19 (Weekly Plan History) | Straightforward | Paginated reads over retained history with safety re-check on reuse (FEAT-19.SPEC-003 ## Processing Logic) |
| FEAT-20 (Online Grocery Ordering Handoff) | Research-spike recommended | No UK self-service grocer API was found and Instacart covers the US only (FEAT-20.SPEC-002 ## Data Exchanged; technology-landscape.md Online Grocery Ordering Integration) |
| FEAT-21 (Family Calendar Sync) | Standard-with-integration | OAuth calendar connection and per-night entry sync with silent retries (FEAT-21.SPEC-002 ## Degradation Behavior; FEAT-21.SPEC-004 ## Business Rules) |
| FEAT-22 (Operator Read-Only Support Access) | Straightforward | Request-scoped, read-only access gating with an append-only access record (FEAT-22.SPEC-004 ## Processing Logic; FEAT-22.SPEC-006, FEAT-22.SPEC-007) |
| FEAT-23 (Manual Weekly Planning) | Straightforward | Per-night picks with reject-with-refresh writes, riding on the shared safety engine and list recalculation (FEAT-23.SPEC-006 ## Edge Cases; FEAT-23.SPEC-004) |
| FEAT-24 (Invite Another Household) | Straightforward | Durable per-member referral link and a write-once attribution record (FEAT-24.SPEC-003 ## Processing Logic; FEAT-24.SPEC-006) |
| FEAT-25 (Weekly Waste & Spend Check-In) | Straightforward | One weekly record per household on a week-boundary schedule plus a simple trend calculation (FEAT-25.SPEC-003 ## Trigger Definition; FEAT-25.SPEC-004) |

## 2. Per-Feature Assessments

### FEAT-01 — Household Setup & Member Profiles

**Verdict:** Standard-with-integration — setup screens, validation and role rules are well-trodden (FEAT-01.SPEC-003..010, FEAT-01.SPEC-014, FEAT-01.SPEC-016), but account confirmation and password recovery depend on an external transactional-email capability with a queued-send degradation contract (FEAT-01.SPEC-017 ## Degradation Behavior).

**Required Capabilities:**
- Account sign-up, sign-in and password recovery with non-disclosing responses (FEAT-01.SPEC-001, FEAT-01.SPEC-002; FEAT-01.SPEC-017 ## Degradation Behavior — "timing differences never disclose whether an account exists")
- Role-based authorization across Organiser, Other Adult Member and kid profiles (FEAT-01.SPEC-016 ## Authorization Rules)
- Parental consent capture before a kid profile exists, recorded as the organiser's affirmative confirmation (FEAT-01.SPEC-007 ## Scope and Non-Goals); kid-profile data minimality enforced at validation (FEAT-01.SPEC-014)
- Dietary-rule capture against a standard allergen list plus free-text extra ingredients up to 80 characters (FEAT-01.SPEC-015 ## Field Validation Rules), with change history retained (feature-overview.md ## Non-Functional Notes)
- Background trigger on hard-rule changes that starts FEAT-02's mid-week re-check and an in-app notice (FEAT-01.SPEC-012; FEAT-01.SPEC-018 ## Channels); free-tier subscription provisioning at setup (FEAT-01.SPEC-011)
- Concurrency: Maya editing from laptop and phone at once resolves last-write-wins per setting, and a removal racing an own-account edit resolves reject-with-refresh (FEAT-01.SPEC-005, FEAT-01.SPEC-006, FEAT-01.SPEC-008 ## Edge Cases; feature-dependency-map.md, Household and Member Profile **Contention:**)
- Offline/degraded: every setup screen keeps a local draft and queues saves offline for automatic submission on reconnect (FEAT-01.SPEC-013 ## Business Rules); allergy removal is blocked offline because it needs a live check (FEAT-01.SPEC-015 ## Edge Cases); email outages never block account creation (FEAT-01.SPEC-017 ## Degradation Behavior)
- Scale: up to 12 member profiles per household across several thousand households, history kept for the life of the account (feature-overview.md ## Non-Functional Notes; ASMP-24); setup screens are exempt from the ASMP-23 heavier targets

**Candidate Approaches:** Identity can come from a managed provider (Clerk, Supabase Auth or Auth0 in the Authentication & Identity area), where Clerk's prebuilt components shorten the setup flow and Supabase Auth ties role checks to Postgres row-level security for kid-profile restrictions; or from a self-hosted library (Better Auth, NextAuth.js), which gives full control over the parental-consent gating and kid-profile no-login model at the cost of owning session hardening. Recovery and confirmation email fits any of Resend, Postmark or SendGrid (Email & Messaging Delivery); Postmark's transactional-only positioning suits time-sensitive recovery links, Resend's free tier suits early volume. Offline drafts map to the Service-worker / IndexedDB client-side caching option (Caching & Performance), with client state held in TanStack Query + Zustand, Redux Toolkit + RTK Query or Jotai (State Management); Jotai's atoms fit per-field draft state. Queued email sends need a durable job runner (Inngest, Trigger.dev, Upstash QStash or BullMQ in Background Jobs & Scheduling).

**Risks & Unknowns:** ASMP-27 states "verifiable parental consent" but FEAT-01.SPEC-007 ## Scope and Non-Goals relies on the organiser's affirmative confirmation with no verification service. Whether that satisfies children's-privacy-class obligations in the US and UK is a legal question, and the landscape has no consent-verification option; if stronger verification is needed, a new capability has to be researched (see Section 5). Offline-queued saves replayed after a server-side validation or authorization change (for example, the organiser role was handed over while a draft was queued) have to surface as a retry state and never be dropped silently (FEAT-01.SPEC-013 ## Business Rules). Dietary Rule data is the most sensitive class in the product, so any managed auth or BaaS vendor that stores profile data falls inside the children's-data processing boundary (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** None

### FEAT-02 — Dietary Rules & Allergy Safety Engine

**Verdict:** Hard — the check has to be exhaustive and fail closed on every path onto the plan (XBR-01). It compares full ingredient text against each member's rules, expands compound terms such as "mixed nuts" to every allergen they could contain, excludes any recipe with incomplete quantity or unit data, and must still fit the seconds-level swap and sub-minute generation budgets as the recipe pool grows, with "no full re-scan shortcuts" (FEAT-02.SPEC-002 ## Processing Logic, ## Edge Cases; FEAT-02.SPEC-007 ## Field Validation Rules; feature-overview.md ## Non-Functional Notes).

**Required Capabilities:**
- Deterministic, stateless per-candidate safety evaluation against every member's hard rules, with a vegetarian-variant exception and plain-language ineligibility reasons (FEAT-02.SPEC-002 ## Processing Logic steps 1–11; FEAT-02.SPEC-006; FEAT-02.SPEC-008)
- An ingredient-to-allergen classification that maps free-text ingredients from starter and imported recipes onto the standard allergen list and named extra ingredients, including compound-term expansion (FEAT-02.SPEC-002 ## Edge Cases; FEAT-01.SPEC-015 ## Field Validation Rules)
- Fail-closed completeness gate: every ingredient needs a name, quantity and unit (FEAT-02.SPEC-007 ## Field Validation Rules)
- Mid-week re-check on rule change that flags and removes failing dinners, offers alternatives and updates the list (FEAT-02.SPEC-003; XBR-02)
- Safety-concern intake that removes the meal, excludes the recipe while the report is open, raises a Support Request and emails the operator (FEAT-02.SPEC-004, FEAT-02.SPEC-005, FEAT-02.SPEC-009; FEAT-02.SPEC-010 ## Degradation Behavior; FEAT-02.SPEC-013 ## Channels — email)
- Concurrency: concurrent checks are read-only and independent (FEAT-02.SPEC-002 ## Edge Cases); a safety removal always wins over any concurrent plan change (feature-dependency-map.md, Weekly Plan and Planned Meal **Contention:**)
- Offline/degraded: the check never runs client-side unchecked, so nothing is shown unchecked when offline. The report screen needs connectivity (FEAT-02.SPEC-001 ## States). Operator email outages leave the Support Request visible in the Support View (FEAT-02.SPEC-010 ## Degradation Behavior)
- Scale: one check per candidate recipe per household on every generation, swap, manual pick, vote and history reuse across several thousand households, and it must stay equally fast as recipe pools grow (feature-overview.md ## Non-Functional Notes; ASMP-22, ASMP-23, ASMP-24)

**Candidate Approaches:** The engine is application logic in whichever Backend / API Layer option is selected (NestJS, Fastify, Express.js, Next.js Route Handlers / Server Actions or Django). Rule and ingredient data sit in a relational store (Neon, Supabase, Amazon RDS for PostgreSQL, PlanetScale for Postgres or self-hosted PostgreSQL in the Database area). Precomputing per-recipe allergen sets at ingest and caching per-household verdicts (Upstash Redis, Redis Cloud or Vercel KV in Caching & Performance) are ways to meet the latency budget without skipping checks. The ingredient-to-allergen taxonomy can come from a content vendor's ingredient data (Spoonacular Food API or Edamam Recipe Search API in Recipe & Food Content Data; Edamam is positioned for nutrition and allergen analysis) or from a curated in-house mapping maintained alongside the Curated/licensed starter set + manual editorial seeding option. Operator alerts go through any Email & Messaging Delivery option (Resend, Postmark, SendGrid).

**Risks & Unknowns:** The engine is only as safe as its ingredient-to-allergen mapping. Imported recipes (FEAT-10) carry arbitrary free text such as "stock", "pesto" and "Worcestershire sauce" whose allergen content is implicit. The landscape has no dedicated allergen-taxonomy option, only content vendors' own taxonomies, and vendor lock-in to an ingredient taxonomy is flagged for Spoonacular and Edamam. Under-matching breaks the "zero allergy incidents, ever" metric (success-metrics.md, cited in FEAT-23 ## Non-Functional Notes). Over-matching and strict quantity/unit completeness could exclude so many recipes that households with several allergies get repetitive or partial plans (FEAT-03.SPEC-010 ## Edge Cases — very small pools). Caching verdicts creates invalidation risk: any rule change, recipe edit or open safety report has to invalidate cached verdicts, or a stale "pass" could leak through.

**Spike Recommendation:** Build an ingredient-to-allergen classifier prototype and run it over a sample of starter-source recipes and pasted-link recipes (for example, 200 recipes across target cuisines). Measure (a) recall against a hand-labeled allergen truth set, (b) the share of recipes excluded by the FEAT-02.SPEC-007 completeness bar, and (c) per-candidate check latency. The result tells the build team whether a vendor taxonomy is enough, a curated mapping is needed, or both are layered, and whether the completeness rule needs a product-level carve-out for "to taste" style ingredients.

### FEAT-03 — AI Weekly Dinner Plan Generation

**Verdict:** Research-spike recommended — the resolvable unknown is whether a managed LLM, given the household's constraints and verified candidate pool (FEAT-03.SPEC-010 ## Data Exchanged), reliably returns a full seven-night selection that passes FEAT-02, fits the budget rule (FEAT-03.SPEC-006) and schedule, in well under a minute (ASMP-23), at a per-household token cost inside the pre-revenue budget (SC-16, as cited in FEAT-04 ## Non-Functional Notes). The specs also define partial-result and failure outcomes whose frequency is unknown (FEAT-03.SPEC-010 ## Edge Cases).

**Required Capabilities:**
- AI plan-generation integration that sends constraints, aggregated ratings, pantry items and the candidate pool, and receives candidate selections plus a coverage signal (FEAT-03.SPEC-010 ## Data Exchanged)
- Scheduled per-household generation at the organiser's chosen arrival day and time, plus immediate first-plan generation on upgrade (FEAT-03.SPEC-003 ## Trigger Definition; FEAT-03.SPEC-004)
- Week-start auto-adoption and once-per-week organiser approval (FEAT-03.SPEC-005; FEAT-03.SPEC-008; XBR-07)
- Budget-fit and household-scaled quantity computations (FEAT-03.SPEC-006, FEAT-03.SPEC-007); tier gating (FEAT-03.SPEC-009)
- Live plan propagation across household devices (FEAT-03.SPEC-011 ## Degradation Behavior)
- Concurrency: only one generation run in flight per household and week (FEAT-03.SPEC-010 ## Edge Cases); approval, auto-adoption, swaps and safety removals all write the plan with reject-with-refresh per slot (feature-dependency-map.md, Weekly Plan **Contention:**); late responses after a failure are discarded (FEAT-03.SPEC-010 ## Edge Cases)
- Offline/degraded: an already-generated plan stays viewable offline and approving needs a reconnect (FEAT-03.SPEC-011 ## Degradation Behavior; ASMP-25). An AI outage keeps the prior week visible with a Retry banner, and slow responses show "This is taking longer than usual" (FEAT-03.SPEC-010 ## Degradation Behavior)
- Scale: one generation per paid household per week, several thousand households, one archived week per household per week kept indefinitely; generation well under a minute; sync in the near-instant window (feature-overview.md ## Non-Functional Notes; ASMP-22, ASMP-23, ASMP-24)

**Candidate Approaches:** The generation call can target Anthropic Claude API, OpenAI GPT API or Google Gemini API (AI & Intelligent Behavior). All three offer structured output. Claude's prompt caching and OpenAI's Batch API (50% cheaper, for the non-interactive scheduled run) are the cost levers that differ, and Gemini Flash is the lowest listed per-token price. A Self-hosted open-weight model is in the landscape but is characterized there as disproportionate to "one weekly plan plus a few swaps". A hybrid is also possible within the same options: deterministic pre-filtering (safety, schedule, budget) shrinks the pool before the LLM arranges it, reducing tokens and failure modes. Scheduling options are Inngest, Trigger.dev (durable, step-based retries), Upstash QStash (HTTP delivery at a set time) or BullMQ (Background Jobs & Scheduling). Live propagation options are Supabase Realtime, Ably, Pusher (Channels) or Socket.io (self-hosted) (Real-time & Collaboration); Ably's ordering guarantees and Pusher's lighter channel model are the differentiators for a 2–6-member household channel.

**Risks & Unknowns:** Constraint-satisfaction quality: LLMs can propose recipes outside the supplied pool, skip nights or ignore budget. The spec covers this by discarding out-of-pool candidates and treating short results as partial (FEAT-03.SPEC-010 ## Edge Cases), but how often that happens drives the failure rate households see. The token budget grows with pool size, because the full candidate pool with ingredient lists is sent on every attempt (FEAT-03.SPEC-010 ## Data Exchanged), so imported recipes (FEAT-10) raise cost per generation. Scheduled bursts: default arrival slots (Sunday evening, platform-parameters.md `plan-arrival-time-slots`) cluster thousands of generations into a few time slots, which exposes provider rate limits. Dietary constraints leave the product (per member, without identifying detail), so the provider's data-retention terms fall under children's-privacy-class handling (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** Run a bounded generation harness with 20–30 synthetic households spanning multiple allergies, vegetarian members, tight budgets and small pools, against two or three of the landscape's LLM options. Measure the full-week success rate after FEAT-02 filtering, p95 latency, tokens per generation and the resulting monthly cost at several thousand paid households, plus behavior under Sunday-evening burst concurrency. That tells the Architect which provider and prompt structure (full-pool vs pre-filtered) meets ASMP-23 and SC-16, and whether a deterministic fallback selector is needed.

### FEAT-04 — One-Tap Meal Swap

**Verdict:** Standard-with-integration — swap alternatives come from the external AI capability with a fully specified degradation contract (FEAT-04.SPEC-007 ## Degradation Behavior). The per-slot lock with first-to-acquire-wins and guaranteed release (FEAT-04.SPEC-009 ## Business Rules) and the suggestion lifecycle (FEAT-04.SPEC-010) are conventional transactional patterns.

**Required Capabilities:**
- AI swap-alternatives generation, filtered through FEAT-02 and ranked with scarcity explanations (FEAT-04.SPEC-007; FEAT-04.SPEC-008)
- Apply a swap atomically: lock, safety re-check, write, grocery-list recalculation and live propagation (FEAT-04.SPEC-004; XBR-03)
- Other adults suggest and the organiser accepts or declines; suggestions lapse when the night passes (FEAT-04.SPEC-002, FEAT-04.SPEC-003, FEAT-04.SPEC-005; XBR-06); in-app suggestion notices (FEAT-04.SPEC-006 ## Channels)
- Concurrency: one active swap per slot, and a second attempt is refused outright, not queued (FEAT-04.SPEC-009 ## Business Rules). Swap Suggestion resolves first-decision-wins, and a direct swap supersedes an open suggestion (feature-dependency-map.md, Swap Suggestion **Contention:**)
- Offline/degraded: an AI outage shows a Retry with the original meal unchanged, and slow responses show a note after 8 seconds (FEAT-04.SPEC-007 ## Degradation Behavior); no half-created suggestion or meal is ever written
- Scale: a few swaps per household per week; tap-to-updated-plan-and-list under 10 seconds; alternatives within a couple of seconds (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Alternatives can come from the same AI & Intelligent Behavior option as FEAT-03 (Anthropic Claude API, OpenAI GPT API, Google Gemini API), where lower-latency model tiers such as Gemini Flash matter more here than in weekly generation. Alternatively, a deterministic ranking over the already-verified candidate pool can serve alternatives without an LLM call, keeping the AI path for explanation or ranking only. The per-slot lock can be a database row lock or conditional write in any Database option, or a short-lived key in Upstash Redis or Redis Cloud (Caching & Performance). Lapse scheduling fits Inngest, Trigger.dev, Upstash QStash or BullMQ (Background Jobs & Scheduling). Propagation reuses the Real-time & Collaboration choice (Supabase Realtime, Ably, Pusher (Channels), Socket.io (self-hosted)).

**Risks & Unknowns:** The under-10-second budget chains an external LLM call, a safety check, a write, list recalculation and cross-device propagation. LLM tail latency alone can consume most of it (FEAT-04.SPEC-007 ## Degradation Behavior already anticipates more than 8 seconds). A lock held in a cache store can outlive a crashed request unless it has a TTL, which conflicts with "never left held" (FEAT-04.SPEC-009 ## Business Rules). Lapse timing depends on the household's local time zone and daylight-saving transitions (FEAT-04.SPEC-005 ## Edge Cases).

**Spike Recommendation:** None — the FEAT-03 spike's latency measurements can include a swap-alternatives prompt to settle whether the couple-of-seconds target needs the deterministic path.

### FEAT-05 — Pantry-Aware Suggestions

**Verdict:** Straightforward — a short, per-household list with exact-name duplicate merge (FEAT-05.SPEC-003) and exact-name recipe matching for callouts and weighting (FEAT-05.SPEC-006 ## Business Rules). Weighting is only an input to FEAT-03's generation (FEAT-05.SPEC-005), and no new external service is involved.

**Required Capabilities:**
- Pantry item entry, clear and "used it up?" prompts (FEAT-05.SPEC-001; FEAT-05.SPEC-002)
- Exact-name matching between pantry items and recipe ingredients for plan callouts and paid-tier weighting (FEAT-05.SPEC-006; FEAT-05.SPEC-005)
- Pantry items excluded from the grocery list on both tiers (FEAT-05.SPEC-007; XBR-04)
- Concurrency: Maya and Sam add and clear items concurrently, including offline; the same item added twice merges and clearing is idempotent (FEAT-05.SPEC-003 ## Edge Cases; feature-dependency-map.md, Pantry Item **Contention:**)
- Offline/degraded: adds are queued locally and synced on reconnect (feature-overview.md ## Non-Functional Notes; FEAT-05.SPEC-001 ## States; ASMP-25)
- Scale: short lists with no fixed cap, several thousand households; adding confirms instantly (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Offline queuing can share the Service-worker / IndexedDB client-side caching implementation built for FEAT-06 (Caching & Performance). Sync can ride the same Real-time & Collaboration channel (Supabase Realtime, Ably, Pusher (Channels), Socket.io (self-hosted)) or plain request/response with TanStack Query + Zustand or Redux Toolkit + RTK Query refetch (State Management), since pantry edits have no 2-second propagation target. Matching is a relational query in any Database option.

**Risks & Unknowns:** Exact-name matching ("spinach" does not match "baby spinach", FEAT-05.SPEC-006 ## Business Rules) is simple to build but may under-deliver the "uses the spinach and feta you already have" promise. That is a product-quality trade-off, not a feasibility blocker.

**Spike Recommendation:** None

### FEAT-06 — Shared Grocery List

**Verdict:** Hard — the list must show a tick or add on another device within 2 seconds (feature-overview.md ## Non-Functional Notes) and stay fully editable offline. Queued changes then reconcile with idempotent ticks, last-write-wins quantity edits by original event time, deletion precedence and duplicate-line merge, all while plan-driven recalculation rewrites plan-derived lines concurrently (FEAT-06.SPEC-005 ## Degradation Behavior; FEAT-06.SPEC-008 ## Cross-Field Rules, ## Edge Cases; feature-dependency-map.md, Grocery List Item **Contention:** "High").

**Required Capabilities:**
- Live list sync across household devices with optimistic local updates (FEAT-06.SPEC-005 ## Degradation Behavior)
- List generation and recalculation from the current plan, with ingredient consolidation and household-scaled quantities, preserving manual items, ticks and "already have it" marks (FEAT-06.SPEC-002; FEAT-06.SPEC-006; XBR-03)
- "Already have it" marks that also add to the pantry (FEAT-06.SPEC-003; XBR-04); week rollover and carry-over (FEAT-06.SPEC-004); manual-item validation and merge (FEAT-06.SPEC-007); access rules (FEAT-06.SPEC-009)
- Aisle grouping and units per household locale (XBR-11, owned by FEAT-16)
- Concurrency: multiple members plus recalculation edit simultaneously. Ticks are idempotent, quantity edits last-write-wins by event time with arrival-order tiebreak, deletion wins over queued edits, and a removed member's queued changes are discarded (FEAT-06.SPEC-008 ## Cross-Field Rules, ## Business Rules, ## Edge Cases)
- Offline/degraded: a persistent offline banner, with every read and write remaining usable and queued for sync (FEAT-06.SPEC-005 ## Degradation Behavior; ASMP-25)
- Scale: several thousand households, one active list each plus retained history; sub-2-second propagation; recalculated items within a couple of seconds (feature-overview.md ## Non-Functional Notes; ASMP-22, ASMP-24)

**Candidate Approaches:** Transport can be Supabase Realtime (Postgres change-data-capture, no separate service if Supabase is the database), Ably (guaranteed ordering, exactly-once delivery and connection recovery, closest to the event-ordering rules), Pusher (Channels) (lighter household-scoped channels; the merge logic is entirely the team's) or Socket.io (self-hosted) (full control over the reconciliation protocol at the cost of owning scaling and reconnection), all in Real-time & Collaboration. The offline queue and local replica map to Service-worker / IndexedDB client-side caching (Caching & Performance), with server-state caching in TanStack Query + Zustand or Redux Toolkit + RTK Query (State Management). Server-side merge rules live in the Backend / API Layer option against any Database option. Hot-path reads can be cached in Upstash Redis or Redis Cloud. The landscape has no dedicated CRDT or local-first sync engine, so these merge semantics are custom application logic whichever transport is selected.

**Risks & Unknowns:** Last-write-wins "by original event time" (FEAT-06.SPEC-008 ## Cross-Field Rules) depends on client clocks, and skewed device clocks can make an older edit win. The spec's arrival-order tiebreak only covers exact ties. Recalculation racing queued offline ticks on lines whose identity changes (FEAT-06.SPEC-008 ## Edge Cases) needs stable line identity across recalculation. Supabase Realtime's documented household-scale connection figures (~200 concurrent on Pro) and Pusher's free-tier limits need checking against in-store usage peaks such as weekend shopping. Service-worker behavior differs across mobile browsers, especially iOS Safari background sync, which affects how reliably "changes sync later" works (ASMP-25).

**Spike Recommendation:** Prototype the offline queue plus the FEAT-06.SPEC-008 merge rules on one or two candidate transports, for example Ably vs Supabase Realtime. Script two devices through offline tick/edit/delete sequences, clock skew, and a concurrent recalculation. The spike answers whether the rules converge with no duplicates on the target mobile browsers and whether server-assigned sequencing needs to replace client event times. That decides the Real-time & Collaboration selection and the sync protocol shape.

### FEAT-07 — Weekly Plan Ready Notification

**Verdict:** Standard-with-integration — delivery depends on two external capabilities, device push (FEAT-07.SPEC-005) and email fallback (FEAT-07.SPEC-006), each with a queued/no-error degradation contract (## Degradation Behavior in both). The at-most-once-per-household-week rule is a conventional idempotency key (FEAT-07.SPEC-002 ## Delivery Rules).

**Required Capabilities:**
- Event-driven dispatch on generation completion (FEAT-07.SPEC-001 ## Trigger Definition) with per-member channel resolution and eligibility (FEAT-07.SPEC-003) and organiser-set arrival day/time (FEAT-07.SPEC-004)
- Web push to a responsive web app, with no native apps (FEAT-07.SPEC-005 ## Scope and Non-Goals, SC-05); email fallback (FEAT-07.SPEC-006)
- Deduplication and expiry: one message per member per household-week, and offline-device messages are discarded when the next week's message is due (FEAT-07.SPEC-002 ## Delivery Rules)
- Concurrency: duplicate or retried completion signals must never double-send (FEAT-07.SPEC-002 ## Delivery Rules); preference edits racing dispatch resolve on the current preference (FEAT-07.SPEC-003)
- Offline/degraded: offline devices get queued push on reconnect. A push outage drops to the email fallback, and an email outage sends nothing and shows no error, because the plan's in-app availability never depends on delivery (FEAT-07.SPEC-005, FEAT-07.SPEC-006 ## Degradation Behavior; XBR-12)
- Scale: at most one message per household per week to 2–6 members across several thousand households; at least 80% delivered within one minute of completion (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Push can go through OneSignal (fastest self-serve Web Push setup) or Courier (one API over push and email with routing, adding an abstraction layer), and email through Resend, Postmark or SendGrid, all in Email & Messaging Delivery. A Courier-style unified layer handles channel fallback centrally, whereas OneSignal + Resend/Postmark keeps fallback logic in the app. Dispatch jobs fit Inngest, Trigger.dev, Upstash QStash or BullMQ (Background Jobs & Scheduling).

**Risks & Unknowns:** Web Push reach on iPhones depends on the user installing the web app to the home screen, and without that, iOS members fall to email. That could put the 80%-within-a-minute target and the "at least half opened same day" metric at risk (feature-overview.md ## Non-Functional Notes). Clustered Sunday-evening arrival slots concentrate dispatch bursts (platform-parameters.md `plan-arrival-time-slots`), so provider throughput and rate limits matter.

**Spike Recommendation:** None

### FEAT-08 — Recipe Library (Starter Recipes)

**Verdict:** Standard-with-integration — browse and search is a simple filter (FEAT-08.SPEC-001; profile Section 3 Search, simple filter), but the library's existence depends on an external recipe/food-content source delivering ingredient-complete content in batches, with incomplete batches held out (FEAT-08.SPEC-004 ## Data Exchanged, ## Edge Cases).

**Required Capabilities:**
- Search by name or ingredient and filter by dietary badge, with ineligible-recipe disclosure (FEAT-08.SPEC-001; FEAT-08.SPEC-003)
- Recipe detail with locale-converted quantities and safety badge (FEAT-08.SPEC-002; XBR-11; FEAT-02.SPEC-008)
- Seeding and maintenance pipeline: new content, corrections, retirements, and a completeness gate before content becomes Active (FEAT-08.SPEC-004 ## Data Exchanged, ## Edge Cases)
- Concurrency: N/A for households, since starter recipes are read-only (feature-dependency-map.md, Recipe **Contention:**); duplicate corrections resolve most-recent-wins (FEAT-08.SPEC-004 ## Edge Cases)
- Offline/degraded: a content-source outage has no household-visible impact (FEAT-08.SPEC-004 ## Degradation Behavior); failed detail loads offer retry without losing search context (feature-overview.md ## Non-Functional Notes)
- Scale: one shared corpus sized to the Recipe Library Coverage at Launch metric; concurrent search from several thousand households with results in about a second (feature-overview.md ## Non-Functional Notes; ASMP-23)

**Candidate Approaches:** Content can come from Spoonacular Food API (~365,000 recipes with ingredient data), Edamam Recipe Search API (nutrition and allergen specialization), a Curated/licensed starter set + manual editorial seeding (highest control over the FEAT-02.SPEC-007 completeness bar), or Tasty/Yummly-class licensed content partnerships (negotiated terms), all in Recipe & Food Content Data. Search at this complexity is satisfiable by PostgreSQL full-text search. Meilisearch, Typesense or Algolia (Search) add typo tolerance at the cost of an index-sync pipeline and, for Algolia, higher cost.

**Risks & Unknowns:** Vendor licensing terms for storing and redisplaying recipe content (as opposed to live API lookups) are not captured in the landscape, and some API tiers restrict caching. The API tier costs listed ($149–$999/month) sit against the founder's sub-$100/month pre-revenue budget (SC-16, cited in FEAT-04 ## Non-Functional Notes). The share of vendor recipes that pass the quantity+unit completeness bar is unknown (see the FEAT-02 spike).

**Spike Recommendation:** None — covered by the FEAT-02 spike's completeness measurement on a starter-source sample; licensing terms are a Section 5 question.

### FEAT-09 — Household Invitations & Membership

**Verdict:** Straightforward — invitation create/resend/revoke, 14-day scheduled expiry, first-write-wins acceptance, organiser hand-over requiring acceptance and member departure are conventional state transitions (FEAT-09.SPEC-006, FEAT-09.SPEC-007 ## Edge Cases, FEAT-09.SPEC-009, FEAT-09.SPEC-010). All notifications are in-app (FEAT-09.SPEC-012..014 ## Channels).

**Required Capabilities:**
- Shareable invitation link, acceptance flow and Member Profile creation that triggers onboarding once (FEAT-09.SPEC-001, FEAT-09.SPEC-002, FEAT-09.SPEC-007; XBR-18)
- Organiser hand-over with acceptance so exactly one organiser exists (FEAT-09.SPEC-003, FEAT-09.SPEC-004, FEAT-09.SPEC-009; XBR-15); leave-household processing with rating anonymization (FEAT-09.SPEC-008; XBR-16)
- Scheduled 14-day invitation expiry (FEAT-09.SPEC-006)
- Live member-list update on acceptance (profile Section 3 Real-time, FEAT-01.SPEC-004)
- Concurrency: acceptance vs revocation vs expiry resolves reject-with-refresh, first recorded wins, and two simultaneous accepts yield one member (FEAT-09.SPEC-007 ## Edge Cases; feature-dependency-map.md, Invitation **Contention:**)
- Offline/degraded: a drafted invitation is held locally until it can be sent (feature-overview.md ## Non-Functional Notes; FEAT-09.SPEC-001 ## States)
- Scale: 1–12 members and multiple outstanding invitations per household; send confirms within a couple of seconds (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Invitation tokens and state live in any Database option, with transactional conditional updates for race resolution. Invitee sign-up flows through the Authentication & Identity option (Clerk, Supabase Auth, Auth0, Better Auth, NextAuth.js), where managed providers differ in how custom invitation tokens attach to sign-up. Expiry fits any Background Jobs & Scheduling option (Inngest, Trigger.dev, Upstash QStash, BullMQ) or a query-time expiry check with no job at all. The live member list reuses the Real-time & Collaboration option.

**Risks & Unknowns:** Hand-over interacts with account deletion and billing ownership. The Subscription is organiser-controlled (FEAT-14), so payment-method ownership across a hand-over has to be traced into the payment integration, and FEAT-09 specs do not cover it.

**Spike Recommendation:** None

### FEAT-10 — Recipe Import from Web Link

**Verdict:** Standard-with-integration — extraction is an external or self-hosted capability with a specified progress/failure contract that falls back to manual entry (FEAT-10.SPEC-005 ## Degradation Behavior). Review-before-save, duplicate detection, the 30-per-week rate limit and the safety re-check on save or edit are conventional (FEAT-10.SPEC-006, FEAT-10.SPEC-007, FEAT-10.SPEC-008).

**Required Capabilities:**
- Fetch and parse a pasted recipe URL into ingredients, steps and cook time for review (FEAT-10.SPEC-001, FEAT-10.SPEC-002, FEAT-10.SPEC-005)
- Manual entry fallback and edit (FEAT-10.SPEC-003, FEAT-10.SPEC-004)
- Duplicate-link detection surfacing the existing recipe (FEAT-10.SPEC-006; XBR-19)
- Per-household rate limit of 30 imports per week with no lifetime cap (FEAT-10.SPEC-007)
- Safety re-check on every save or edit before plan eligibility (FEAT-10.SPEC-008; XBR-19)
- Concurrency: Maya and Sam editing the same imported recipe resolves reject-with-refresh on the second save (FEAT-10.SPEC-004; feature-dependency-map.md, Recipe **Contention:**)
- Offline/degraded: importing requires connectivity. An unreachable extractor shows a failure path with manual entry, and slow extraction keeps the progress indicator until timeout (FEAT-10.SPEC-001 ## States; FEAT-10.SPEC-005 ## Degradation Behavior)
- Scale: up to 30 imports per household per week; extraction in a few seconds; saved recipes searchable within about a second (feature-overview.md ## Non-Functional Notes; ASMP-23)

**Candidate Approaches:** Parsing options are recipe-scrapers (an open-source library that parses schema.org JSON-LD, Microdata and OpenGraph, Python-based) or a Custom JSON-LD/schema.org parser (self-built, in the backend language). A Managed recipe-extraction API (e.g., Apify recipe-scraper actors) moves site-specific maintenance to a marketplace vendor, and AI-fallback extraction (JSON-LD/Microdata/heuristic parsing + LLM fallback) covers pages without structured markup using the AI & Intelligent Behavior provider. All four are in Web Page Recipe Extraction. recipe-scrapers is Python while most Backend / API Layer candidates are Node, so that pairing may need a separate worker. Extraction jobs and timeouts fit any Background Jobs & Scheduling option. Rate-limit counters fit Upstash Redis or Redis Cloud, or a database counter.

**Risks & Unknowns:** Legality of importing other sites' content is an unresolved product and legal question that gates v1 (ASMP-36; BRIEF.md Open Questions). Server-side fetching of arbitrary user-supplied URLs is a server-side request forgery surface that needs egress restrictions. Many sites block scrapers or render recipes client-side. Extracted ingredient lines often lack a quantity or unit ("salt to taste"), which the FEAT-02.SPEC-007 fail-closed completeness bar excludes, so a successfully imported recipe may never be plan-eligible without manual edits. The AI-fallback option sends third-party page content to an LLM provider, which adds per-import cost variance.

**Spike Recommendation:** Run a bounded extraction test on about 100 links from popular US and UK recipe sites with one self-built/library parser and one managed option. Measure the extraction success rate, the share that pass FEAT-02.SPEC-007 without edits, and median latency against the "few seconds" target. The result tells the team whether an AI-fallback tier is needed and how often the review screen must prompt for missing quantities.

### FEAT-11 — Leftover Rollover to Lunches

**Verdict:** Straightforward — leftover-lunch suggestions are a deterministic pass over the newly generated plan: one-day default, two-day fallback, no suggestion past the XBR-10 ceiling (FEAT-11.SPEC-002 ## Processing Logic; FEAT-11.SPEC-003), with withdrawal when the source dinner changes (FEAT-11.SPEC-004).

**Required Capabilities:**
- Post-generation computation linking each leftover lunch to exactly one source dinner (FEAT-11.SPEC-002; FEAT-11.SPEC-003; XBR-10)
- Update or withdrawal on source swap or removal (FEAT-11.SPEC-004); eaten/skipped marking on the leftover card (FEAT-11.SPEC-001)
- Concurrency: Maya and Sam marking a leftover lunch eaten or skipped resolves last-write-wins, and a source swap racing a mark resolves via withdrawal (FEAT-11.SPEC-001 ## Edge Cases; feature-dependency-map.md, Planned Meal **Contention:**)
- Offline/degraded: an already-generated suggestion stays viewable offline, and a computation failure omits the suggestion silently (feature-overview.md ## Non-Functional Notes)
- Scale: at most one leftover lunch per eligible dinner per week per household; no separate loading step (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Runs as a step inside the generation workflow in any Background Jobs & Scheduling option (Inngest and Trigger.dev model it as a durable step; BullMQ or Upstash QStash as a chained job), or synchronously in the Backend / API Layer option after generation completes. Offline viewing reuses the Service-worker / IndexedDB client-side caching layer.

**Risks & Unknowns:** None identified — no external dependency or scale demand; correctness depends on FEAT-04 and FEAT-23 reliably emitting source-change events.

**Spike Recommendation:** None

### FEAT-12 — Meal Rating & Preference Learning

**Verdict:** Straightforward — rating capture is simple CRUD with proxy rules (FEAT-12.SPEC-002). "Learning" is a counted-threshold update to a soft dislike (FEAT-12.SPEC-005 ## Processing Logic) plus per-recipe aggregated weighting passed into FEAT-03's generation (FEAT-12.SPEC-004 ## Business Rules), so no model training or new external service is specified.

**Required Capabilities:**
- Post-dinner thumbs up/down prompt, with adults recording young kids' ratings by proxy (FEAT-12.SPEC-001; FEAT-12.SPEC-002)
- Repeated-dislike threshold (`repeated-dislike-rating-count`) that creates a learned soft dislike, tier-gated (FEAT-12.SPEC-005 ## Processing Logic; XBR-17)
- Aggregated-per-recipe weighting input for paid-tier generation that never overrides safety (FEAT-12.SPEC-004 ## Business Rules; FEAT-03.SPEC-010 ## Data Exchanged)
- Authorization so individual ratings are never shown broken out (FEAT-12.SPEC-003)
- Concurrency: two adults recording the same kid's rating resolves last-write-wins, and learned dislikes merge with explicit rules without overwriting them (feature-dependency-map.md, Rating and Dietary Rule **Contention:**)
- Offline/degraded: a failed submission retries automatically in the background (feature-overview.md ## Non-Functional Notes; FEAT-12.SPEC-001 ## States)
- Scale: one rating per member per planned meal across several thousand households, kept for life (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Ratings and threshold counts live in any Database option. The learned-update trigger runs inline in the Backend / API Layer option or as an event in Inngest, Trigger.dev or BullMQ (Background Jobs & Scheduling). The weighting's effect on plans depends on the AI & Intelligent Behavior option selected for FEAT-03 (Anthropic Claude API, OpenAI GPT API, Google Gemini API). Background retry of submissions maps to the Service-worker / IndexedDB client-side caching queue.

**Risks & Unknowns:** Whether an LLM actually honors "weight toward highly rated" inputs measurably is part of the FEAT-03 spike's unknown. Rating deletion vs anonymization on member removal or departure (XBR-16) needs careful data modeling. Young-kid ratings are children's data (feature-overview.md ## Non-Functional Notes), and aggregated ratings leave the product in generation requests.

**Spike Recommendation:** None

### FEAT-13 — Tonight's Dinner Reminder

**Verdict:** Standard-with-integration — the nudge is delivered by push through the shared device-notification integration (FEAT-13.SPEC-002 ## Channels, via FEAT-07.SPEC-005), triggered daily per household at 4:00 pm local time (FEAT-13.SPEC-001 ## Trigger Definition; platform-parameters.md `nightly-nudge-send-time`), with at most one same-day correction (FEAT-13.SPEC-003, FEAT-13.SPEC-004; XBR-09).

**Required Capabilities:**
- Daily scheduled trigger evaluated in each household's local time zone (FEAT-13.SPEC-001 ## Trigger Definition, ## Edge Cases)
- Eligibility and channel resolution per member; kids and the operator are never recipients (FEAT-13.SPEC-005; XBR-13)
- Prep-reminder derivation from recipe prep requirements (FEAT-13.SPEC-006)
- Same-day correction after a swap, sent at most once (FEAT-13.SPEC-003; XBR-09)
- Concurrency: a swap landing while the nudge is dispatching needs a deterministic "sent before or after" cut so the correction fires at most once (FEAT-13.SPEC-003 ## Edge Cases)
- Offline/degraded: push to offline devices queues per the FEAT-07.SPEC-005 contract (FEAT-13.SPEC-002 ## Delivery Rules)
- Scale: at most one nudge and one correction per household per day across several thousand households; near-instant delivery at the chosen time (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Per-household local-time firing can be a timezone-bucketed cron fan-out or per-household scheduled events in Inngest, Trigger.dev, Upstash QStash or BullMQ (Background Jobs & Scheduling). QStash and BullMQ differ mainly in per-message pricing vs flat Redis cost at several thousand daily messages. Delivery goes through the Email & Messaging Delivery push option shared with FEAT-07 (OneSignal or Courier).

**Risks & Unknowns:** The same iOS Web Push install dependency as FEAT-07 applies. Daily volume (several thousand households × 365 days) can exceed free tiers of per-run-priced job platforms (Inngest 25,000 runs/month, QStash 500 messages/day), which affects cost. Daylight-saving transitions must not double-fire or skip the nudge.

**Spike Recommendation:** None

### FEAT-14 — Subscription & Billing Management

**Verdict:** Standard-with-integration — the paid subscription depends on an external payment processor that emits upgrade, renewal and retry outcome events (FEAT-14.SPEC-009 ## Inbound Events) under a slow/down/rejects contract (## Degradation Behavior). Grace-period and refund logic (FEAT-14.SPEC-005, FEAT-14.SPEC-007) is established subscription-billing work.

**Required Capabilities:**
- Tier overview, upgrade (monthly/yearly), billing and payment management, downgrade or cancel at period end (FEAT-14.SPEC-001..004)
- Inbound processor events that drive billing_state, including a 7-day grace window after a failed renewal (FEAT-14.SPEC-009 ## Inbound Events; FEAT-14.SPEC-007; FEAT-14.SPEC-005)
- Applying subscription changes that switch tier gating for FEAT-03, FEAT-05 and FEAT-12 (FEAT-14.SPEC-008; XBR-05)
- In-app plus email billing confirmations and grace notices (FEAT-14.SPEC-010, FEAT-14.SPEC-011; FEAT-14.SPEC-012 ## Degradation Behavior)
- Billing amounts in the household currency, at least USD and GBP (feature-overview.md ## Non-Functional Notes; ASMP-28)
- Concurrency: organiser changes racing processor events resolve reject-with-refresh against stale billing state (feature-dependency-map.md, Subscription **Contention:**; FEAT-14.SPEC-005 and FEAT-14.SPEC-007 ## Edge Cases)
- Offline/degraded: viewing the tier works offline and changes need connectivity. A processor outage disables Subscribe with a message and queues grace retries without shortening the window (FEAT-14.SPEC-009 ## Degradation Behavior)
- Scale: one subscription per household, several thousand households; upgrade confirms within a few seconds (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Stripe Billing provides payment and subscription primitives, with the team building the grace-period and refund logic. Paddle acts as merchant of record and absorbs US sales-tax and UK VAT handling at a higher per-transaction rate. Chargebee or Recurly on top of Stripe express dunning and grace rules as configuration. All four are in Payments & Billing. Webhook ingestion runs in the Backend / API Layer option, with durable processing in Inngest, Trigger.dev or BullMQ (Background Jobs & Scheduling). Email delivery maps to Resend, Postmark or SendGrid.

**Risks & Unknowns:** Webhook delivery is at-least-once and can arrive out of order, so billing-state transitions have to be idempotent and ordering-tolerant (FEAT-14.SPEC-009 ## Inbound Events). The spec's 7-day grace window has to be reconciled with the processor's own retry schedule, since the two can diverge. Selling in two tax jurisdictions affects the merchant-of-record vs processor choice. The founder's three-month revenue timeline (ASMP-33) favors lower-integration options.

**Spike Recommendation:** None

### FEAT-15 — Member Onboarding

**Verdict:** Straightforward — a once-per-acceptance routing automation (FEAT-15.SPEC-002, FEAT-15.SPEC-003) to a landing view that reuses existing plan and list surfaces (FEAT-15.SPEC-001), with no stored records of its own (feature-overview.md ## Non-Functional Notes).

**Required Capabilities:**
- Route a newly accepted member to the plan and list together, with an explained empty state (FEAT-15.SPEC-001; FEAT-15.SPEC-002 ## Edge Cases)
- Once-only rule per acceptance; a re-invited former member onboards again (FEAT-15.SPEC-003; XBR-18)
- Signal emission (member_onboarding_started/completed/shown_empty_household; feature-overview.md ## Non-Functional Notes)
- Concurrency: simultaneous acceptances by different invitees run independently, and reloads do not re-trigger (FEAT-15.SPEC-002 ## Edge Cases)
- Offline/degraded: the landing inherits the plan and list offline behavior (FEAT-15.SPEC-001 ## States; ASMP-25)
- Scale: N/A — single per-member event with no growing dataset (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Routing lives in the Frontend Framework option (Next.js, Nuxt 3, SvelteKit, Remix, Astro). Signals go to PostHog, Mixpanel or Amplitude (Analytics & Product Telemetry), where PostHog's self-host option matters if household-linked events must stay in-house.

**Risks & Unknowns:** None identified — no external dependency; correctness depends on FEAT-09.SPEC-007's single "acceptance succeeded" signal.

**Spike Recommendation:** None

### FEAT-16 — Units, Currency & Locale Configuration

**Verdict:** Straightforward — conversion uses fixed factors (1 cup → 240 ml, 1 oz → 28 g, and so on) at display time and never rewrites stored values. Currency is a display label with no exchange-rate conversion (FEAT-16.SPEC-004 ## Cross-Field Rules, ## Scope and Non-Goals).

**Required Capabilities:**
- Household unit system, currency and aisle-name settings with validation and defaults (FEAT-16.SPEC-001, FEAT-16.SPEC-002, FEAT-16.SPEC-003)
- Display-time conversion applied consistently across plan, recipes, list, budget and check-in (FEAT-16.SPEC-004; XBR-11)
- Concurrency: Maya on two devices resolves last-write-wins per setting, and a failed save keeps the prior value (feature-dependency-map.md, Household **Contention:**)
- Offline/degraded: settings screens follow the product's standard offline messaging (FEAT-16.SPEC-001 ## States); conversion is local and needs no connectivity
- Scale: negligible — one locale record per household (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Formatting options are next-intl (Next.js-native), react-i18next, FormatJS (react-intl) (ICU currency/number formatting) or LinguiJS (Internationalization). The fixed conversion table is plain application code shared by client and server.

**Risks & Unknowns:** Fixed volume-to-volume and weight-to-weight factors are specified, but the specs define no volume↔weight conversion (cups to grams for solids). Recipes authored in cups will show millilitres, not grams, under the metric setting. This is a product expectation point, not a build risk.

**Spike Recommendation:** None

### FEAT-17 — Older-Kid Dinner Voting

**Verdict:** Straightforward — rounds of 2–3 safety-validated options with a deterministic tally, unanimous or split resolution, and a fallback at the resolution point (FEAT-17.SPEC-004, FEAT-17.SPEC-005 ## Processing Logic). The only new platform demand is an older-kid limited-login role (FEAT-17.SPEC-006 ## Field Validation Rules).

**Required Capabilities:**
- Voting round setup with options checked by FEAT-02 (FEAT-17.SPEC-002; FEAT-17.SPEC-004; XBR-01)
- Vote casting by older-kid limited logins, tally and resolution (FEAT-17.SPEC-001, FEAT-17.SPEC-003, FEAT-17.SPEC-005)
- A limited-login identity for minors holding children's-privacy-class data (feature-overview.md ## Non-Functional Notes; ASMP-27)
- Concurrency: a vote cast after Maya resolves the round is rejected with refresh (feature-dependency-map.md, Dinner Vote **Contention:**; FEAT-17.SPEC-003 ## Edge Cases)
- Offline/degraded: failed vote submissions retry automatically (feature-overview.md ## Non-Functional Notes; FEAT-17.SPEC-001 ## States)
- Scale: trivial — one open round per night, one vote per older kid (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** The limited-login role can be modeled in any Authentication & Identity option. Clerk and Auth0 support custom roles and restricted sessions as managed features. Supabase Auth pairs roles with row-level security. Better Auth and NextAuth.js leave the kid-session model entirely to the team. Tally and resolution are Backend / API Layer logic; the resolution-point fallback fits any Background Jobs & Scheduling option.

**Risks & Unknowns:** Accounts for minors raise the children's-privacy bar (ASMP-27). Whether a managed identity provider may hold minors' credentials under its terms, and what consent the Later-phase login requires, are open (BRIEF.md Open Questions on kids' representation).

**Spike Recommendation:** None

### FEAT-18 — Account & Data Management

**Verdict:** Standard-with-integration — export generation produces a downloadable file that needs object storage and an email or in-app ready signal (FEAT-18.SPEC-006 ## Processing Logic; FEAT-18.SPEC-012 ## Degradation Behavior). Household deletion is a durable cascade that must finish within 30 days and reach external services such as the calendar disconnect (FEAT-18.SPEC-008 ## Processing Logic step 5).

**Required Capabilities:**
- Export compiling every household record into one readable file, with automatic retry, a download link and a ready notification (FEAT-18.SPEC-001, FEAT-18.SPEC-006, FEAT-18.SPEC-013)
- Member removal, household deletion and own-account deletion cascades (FEAT-18.SPEC-007, FEAT-18.SPEC-008, FEAT-18.SPEC-009; XBR-15, XBR-16), including immediate sign-out of every member (FEAT-18.SPEC-008 step 3)
- Contact support with an email acknowledgement (FEAT-18.SPEC-005; FEAT-18.SPEC-015 ## Channels — email)
- Transactional email for export-ready, deletion-completed and support acknowledgement (FEAT-18.SPEC-012)
- Concurrency: removal racing a member's own edits or offline queue resolves in favor of removal (FEAT-18.SPEC-007 ## Edge Cases; FEAT-06.SPEC-008 ## Edge Cases), and deletion supersedes in-flight edits (feature-dependency-map.md, Household **Contention:**)
- Offline/degraded: email outages leave in-app states unaffected and queue sends (FEAT-18.SPEC-012 ## Degradation Behavior); export and deletion show progress, never an indefinite wait (feature-overview.md ## Non-Functional Notes)
- Scale: exports compile years of history per household and stay reasonably fast; deletion completes within 30 days (feature-overview.md ## Non-Functional Notes; ASMP-24, ASMP-27)

**Candidate Approaches:** Export files can go in Cloudflare R2 (zero egress), Amazon S3 (lifecycle policies for auto-expiry), Supabase Storage (bundled if Supabase is the database) or Backblaze B2 (File & Object Storage). Multi-step export and cascade processing fits durable-step platforms (Inngest, Trigger.dev) or BullMQ workers (Background Jobs & Scheduling). Trigger.dev's self-host option bears on children's-data residency. Emails go through Resend, Postmark or SendGrid.

**Risks & Unknowns:** "Deleted within 30 days" has to reach copies held outside the primary database: processor-held payment data (FEAT-14), push subscriptions, email-provider logs, analytics events, cached verdicts, database backups and previously generated export files. None of these is enumerated in FEAT-18.SPEC-008. The export file concentrates the most sensitive data, including kids' allergy data (feature-overview.md ## Non-Functional Notes), so download links need expiry and access control. Session revocation within a short window depends on the Authentication & Identity option's revocation support.

**Spike Recommendation:** None

### FEAT-19 — Weekly Plan History

**Verdict:** Straightforward — history browse and detail are paginated reads over retained Weekly Plan and Grocery List records (FEAT-19.SPEC-001, FEAT-19.SPEC-002). Reuse re-runs the shared safety check per archived meal against today's rules (FEAT-19.SPEC-003 ## Processing Logic).

**Required Capabilities:**
- Browse past weeks without a depth cap; past-week detail with plan and list (FEAT-19.SPEC-001, FEAT-19.SPEC-002)
- Reuse a past week into a future week with a per-meal safety re-check (FEAT-19.SPEC-003; XBR-01); access and reuse authorization (FEAT-19.SPEC-004)
- Concurrency: reuse writes into a future week follow the Weekly Plan reject-with-refresh rule (feature-dependency-map.md, Weekly Plan **Contention:**; FEAT-19.SPEC-001 ## Edge Cases)
- Offline/degraded: history screens show standard offline messaging; reuse needs connectivity (FEAT-19.SPEC-001 ## States)
- Scale: years of weekly history per household, with past weeks loading within a couple of seconds as history grows (feature-overview.md ## Non-Functional Notes; ASMP-23, ASMP-24)

**Candidate Approaches:** Indexed, household-scoped pagination in any Database option (Neon, Supabase, Amazon RDS for PostgreSQL, PlanetScale for Postgres, self-hosted PostgreSQL) through the ORM / Data Access option (Prisma ORM, Drizzle ORM, TypeORM). Client-side caching of browsed weeks via TanStack Query + Zustand or Redux Toolkit + RTK Query.

**Risks & Unknowns:** None identified — volumes are per-household and small (about 52 weeks a year).

**Spike Recommendation:** None

### FEAT-20 — Online Grocery Ordering Handoff

**Verdict:** Research-spike recommended — the resolvable unknown is which grocery-ordering partners can actually receive a list payload and return acceptance and item-availability for both launch markets. Instacart Connect / IDP covers the US only, and no self-service Tesco or UK grocer API was found, so the landscape flags UK coverage as a research gap (FEAT-20.SPEC-002 ## Data Exchanged; technology-landscape.md Online Grocery Ordering Integration).

**Required Capabilities:**
- Snapshot of unticked items sent to an external ordering capability, with acceptance/rejection and availability responses handled (FEAT-20.SPEC-002 ## Data Exchanged, ## Edge Cases)
- Eligibility and authorization rules for handoff (FEAT-20.SPEC-003)
- Signals grocery_handoff_initiated, grocery_handoff_succeeded and grocery_handoff_failed (feature-overview.md ## Non-Functional Notes)
- Concurrency: duplicate or out-of-order response events for one attempt are ignored after the first, and responses for abandoned attempts are discarded (FEAT-20.SPEC-002 ## Edge Cases)
- Offline/degraded: when the capability is down, nothing is sent and the list stays unchanged with Retry; slow responses show a "Still working" note (FEAT-20.SPEC-002 ## Degradation Behavior; FEAT-20.SPEC-001 ## States)
- Scale: N/A — no entity of its own; payload bounded by one grocery list (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** US coverage can come from Instacart Connect / Instacart Developer Platform (IDP) with self-service keys. UK coverage can come from Grocery aggregator APIs (e.g., listed on API marketplaces), each to be evaluated, or from a Direct retailer partnership/custom integration (e.g., a UK grocer such as Tesco) through business development (Online Grocery Ordering Integration). The options differ in onboarding path (self-serve vs negotiated) and market reach.

**Risks & Unknowns:** No UK partner may be reachable on reasonable terms, which would make the feature US-only. Mapping free-text list lines to retailer catalog SKUs (units, pack sizes) is not specified upstream. The Grocery List leaves the product boundary here (feature-overview.md ## Non-Functional Notes), which needs a data-sharing disclosure. Later phase, so no MVP impact (ASMP-37).

**Spike Recommendation:** Run a time-boxed partner discovery before the Later phase. Confirm Instacart IDP's commercial terms and its list-to-cart item-matching behavior. Contact at least two UK grocers or aggregators about API availability. Test whether list lines like "2 onions" resolve to catalog items at an acceptable match rate. The answer settles the launch market set and the integration shape for FEAT-20.SPEC-002.

### FEAT-21 — Family Calendar Sync

**Verdict:** Standard-with-integration — an external calendar capability with an OAuth-style connect/disconnect (FEAT-21.SPEC-001; FEAT-21.SPEC-002 ## Degradation Behavior) and background per-night create/update with silent per-night retries, additive-only and non-blocking (FEAT-21.SPEC-003; FEAT-21.SPEC-004 ## Business Rules).

**Required Capabilities:**
- Connect and disconnect one calendar per household (FEAT-21.SPEC-001; FEAT-21.SPEC-004)
- Background per-night entry create/update from the current plan, content limited to dish and night (FEAT-21.SPEC-003; FEAT-21.SPEC-005)
- Inbound sync-outcome and disconnect events (FEAT-21.SPEC-002)
- Concurrency: plan changes during sync, and connect vs disconnect, resolve under the one-connection rule with per-night retry counters (FEAT-21.SPEC-004 ## Business Rules)
- Offline/degraded: sync failures are silent and retried; a failed connect returns to Not Connected with the plan unaffected (FEAT-21.SPEC-002 ## Degradation Behavior)
- Scale: at most one entry per planned meal per connected household, one week ahead (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Google Calendar API (direct) for a single provider. Cronofy or Nylas as unified APIs across Google, Microsoft and Apple (Nylas bundles more scope than needed, and the landscape flags a forced-migration lock-in signal). Direct multi-provider integration (Google Calendar API + Microsoft Graph API, self-built). All four are in Family Calendar Integration. Retries fit any Background Jobs & Scheduling option.

**Risks & Unknowns:** Google's OAuth verification for calendar-write scopes can add review lead time. Apple/iCloud calendar has no first-party REST API outside the unified vendors. Later phase (ASMP-37). The household-deletion cascade must disconnect the calendar (FEAT-18.SPEC-008 step 5).

**Spike Recommendation:** None

### FEAT-22 — Operator Read-Only Support Access

**Verdict:** Straightforward — access is gated to one household with an open Support Request (FEAT-22.SPEC-006). The view is read-only with kid-profile and payment fields masked (FEAT-22.SPEC-007), and every open and close appends to an access record visible to the organiser (FEAT-22.SPEC-004 ## Processing Logic; FEAT-22.SPEC-009).

**Required Capabilities:**
- Support request queue and read-only household view for the operator (FEAT-22.SPEC-001, FEAT-22.SPEC-002)
- Scope gating tied to request status, with field-level visibility rules (FEAT-22.SPEC-006, FEAT-22.SPEC-007; XBR-14)
- Append-only access-session logging and status transitions (FEAT-22.SPEC-004, FEAT-22.SPEC-005, FEAT-22.SPEC-008); in-app household notices (FEAT-22.SPEC-009, FEAT-22.SPEC-010 ## Channels)
- Concurrency: only one household open at a time for Riley, and request resolution closes access (FEAT-22.SPEC-002 ## Edge Cases; feature-dependency-map.md, Support Request **Contention:** None)
- Offline/degraded: standard offline messaging on operator screens (FEAT-22.SPEC-002 ## States)
- Scale: rare, on-demand, one household at a time (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** An operator role in the Authentication & Identity option (Clerk, Auth0 or Supabase Auth roles; Better Auth or NextAuth.js custom roles). Enforcement at the data layer (Supabase row-level security policies, or query-scoped repositories via Prisma ORM or Drizzle ORM) vs the API layer (NestJS guards and similar in the Backend / API Layer area). Audit records in any Database option.

**Risks & Unknowns:** Read-only must be enforced server-side, not just by hiding UI. Observability tools (Sentry session replay, Datadog RUM in Observability & Operations) can capture masked kid data from the operator's screens unless scrubbed.

**Spike Recommendation:** None

### FEAT-23 — Manual Weekly Planning

**Verdict:** Straightforward — per-night pick, change and clear with reject-with-refresh on stale night state (FEAT-23.SPEC-006 ## Edge Cases) and safe-choice filtering through the shared safety engine (FEAT-23.SPEC-004). List updates and live propagation reuse FEAT-06 and FEAT-03's sync, and no AI is involved (feature-overview.md ## Non-Functional Notes).

**Required Capabilities:**
- Week builder, recipe picker and search within about a second (FEAT-23.SPEC-001, FEAT-23.SPEC-002; ASMP-23)
- Placement blocked for ineligible recipes, with reasons in words (FEAT-23.SPEC-004; XBR-01)
- Pick suggestions from other adults (FEAT-23.SPEC-003; XBR-06); validation limits (FEAT-23.SPEC-005)
- Apply a pick and trigger list recalculation (FEAT-23.SPEC-006; XBR-03)
- Concurrency: stale night state is rejected ("This night changed while you were choosing"), a safety removal wins, and picks on different nights proceed independently (FEAT-23.SPEC-006 ## Edge Cases)
- Offline/degraded: failed saves stay visible with retry (FEAT-23.SPEC-006 ## Edge Cases); the plan stays viewable offline (ASMP-25)
- Scale: zero AI cost; weekly plan growth equal to FEAT-03's; picks instant once reflected on the list (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Picker search via PostgreSQL full-text search or Meilisearch, Typesense or Algolia (Search). Conditional writes per night slot in any Database option. Propagation via the shared Real-time & Collaboration option (Supabase Realtime, Ably, Pusher (Channels), Socket.io (self-hosted)).

**Risks & Unknowns:** Inherits FEAT-02's latency risk, because every picker listing needs per-recipe verdicts for the household. Without precomputed verdicts, filtering the whole library on each search could miss the one-second target (ASMP-23).

**Spike Recommendation:** None

### FEAT-24 — Invite Another Household

**Verdict:** Straightforward — one durable, reusable referral link per adult member (FEAT-24.SPEC-003 ## Processing Logic), a write-once attribution record created at setup completion within 30 days (FEAT-24.SPEC-004; FEAT-24.SPEC-006; XBR-20), and an upgrade flag derived from the new household's subscription (FEAT-24.SPEC-005).

**Required Capabilities:**
- Personal referral link provisioning and a share screen (FEAT-24.SPEC-001, FEAT-24.SPEC-003)
- Referral welcome screen that shows only the inviter's first name to visitors (FEAT-24.SPEC-002)
- Attribution captured through sign-up and setup, and upgrade tracking (FEAT-24.SPEC-004, FEAT-24.SPEC-005); in-app referral-joined notice (FEAT-24.SPEC-007 ## Channels)
- Concurrency: link provisioning is idempotent, returning the same link from any device (FEAT-24.SPEC-003 ## Processing Logic); the referral record is never edited (feature-dependency-map.md, Household Referral **Contention:** None)
- Offline/degraded: link creation needs connectivity, and an existing link is shown from local state (FEAT-24.SPEC-001 ## States; FEAT-24.SPEC-003 ## Edge Cases)
- Scale: roughly one referral record per new household, several thousand in year one (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Attribution can travel through the sign-up flow of the Authentication & Identity option (custom metadata in Clerk, Auth0 or Supabase Auth; a custom parameter in Better Auth or NextAuth.js). Growth measurement can go to PostHog, Mixpanel or Amplitude (Analytics & Product Telemetry) alongside the in-database record.

**Risks & Unknowns:** Attribution can break if the 30-day window crosses devices (link opened on one device, setup completed on another). That is a measurement-accuracy risk, not a feasibility one.

**Spike Recommendation:** None

### FEAT-25 — Weekly Waste & Spend Check-In

**Verdict:** Straightforward — one optional record per household per week created on the plan's week boundary (FEAT-25.SPEC-003 ## Trigger Definition), with a simple trend against a starting-point record (FEAT-25.SPEC-004) and validation and access rules (FEAT-25.SPEC-005).

**Required Capabilities:**
- Weekly check-in card and trend view (FEAT-25.SPEC-001, FEAT-25.SPEC-002)
- Week-boundary cycle trigger per household (FEAT-25.SPEC-003)
- Spend shown in household currency (XBR-11)
- Concurrency: two adults answering the same week's check-in resolves under FEAT-25.SPEC-005's single-record rule, and cycle firing is once per household per week (FEAT-25.SPEC-003 ## Edge Cases)
- Offline/degraded: the card follows the product's standard offline messaging (FEAT-25.SPEC-001 ## States); ASMP-25's live-list resilience explicitly does not apply (feature-overview.md ## Non-Functional Notes)
- Scale: about 52 records per household per year (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** The cycle can be a lazily created record when the card first renders in a new week (no job needed) or a scheduled job in any Background Jobs & Scheduling option (Inngest, Trigger.dev, Upstash QStash, BullMQ). Cross-household success aggregation can go to PostHog, Mixpanel or Amplitude.

**Risks & Unknowns:** None identified — trivial volume, no external dependency.

**Spike Recommendation:** None

## 3. Cross-Feature Technical Themes

| Theme / Shared Subsystem | Features Involved | Evidence That Makes It Shared |
|--------------------------|-------------------|-------------------------------|
| App-enforced safety check on every path onto the plan | FEAT-02, FEAT-03, FEAT-04, FEAT-08, FEAT-10, FEAT-17, FEAT-19, FEAT-23 | XBR-01; generation (FEAT-03.SPEC-010 ## Data Exchanged "only after passing FEAT-02's safety check"), swaps (FEAT-04.SPEC-008), badges on browse (FEAT-08.SPEC-003), import save/edit re-check (FEAT-10.SPEC-008), vote options (FEAT-17.SPEC-004), history reuse (FEAT-19.SPEC-003 ## Processing Logic), manual picks (FEAT-23.SPEC-004) all invoke FEAT-02.SPEC-002 |
| AI generation integration and cost envelope | FEAT-03, FEAT-04, FEAT-05, FEAT-12, FEAT-10 | Plan generation (FEAT-03.SPEC-010) and swap alternatives (FEAT-04.SPEC-007) share the ASMP-30 capability; pantry (FEAT-05.SPEC-005) and rating (FEAT-12.SPEC-004) weighting are inputs to the same call; the landscape's AI-fallback extraction option for FEAT-10 depends on the same provider (technology-landscape.md Section 3) |
| Real-time propagation across household devices | FEAT-03, FEAT-04, FEAT-06, FEAT-09, FEAT-23 | FEAT-03.SPEC-011 and FEAT-06.SPEC-005 Integration specs; FEAT-04 and FEAT-23 rely on them (feature-dependency-map.md External Touchpoints, ASMP-35); live member list on acceptance (profile Section 3 Real-time, FEAT-01.SPEC-004) |
| Offline local persistence and queued sync | FEAT-01, FEAT-03, FEAT-05, FEAT-06, FEAT-09, FEAT-11, FEAT-12, FEAT-17 | Setup drafts (FEAT-01.SPEC-013), offline plan viewing (FEAT-03.SPEC-011 ## Degradation Behavior), pantry adds (FEAT-05 ## Non-Functional Notes), list queue (FEAT-06.SPEC-008), held invitation drafts (FEAT-09 ## Non-Functional Notes), leftover viewing (FEAT-11 ## Non-Functional Notes), rating and vote background retries (FEAT-12.SPEC-001, FEAT-17.SPEC-001 ## States) |
| Shared-record contention (reject-with-refresh, last-write-wins, merge) | FEAT-01, FEAT-02, FEAT-03, FEAT-04, FEAT-06, FEAT-09, FEAT-10, FEAT-11, FEAT-12, FEAT-14, FEAT-17, FEAT-23 | 16 of 17 entities carry **Contention:** rules in feature-dependency-map.md ## Shared Data Entities; Weekly Plan/Planned Meal writes from FEAT-02/03/04/11/23 share the per-slot reject-with-refresh and safety-removal-wins rules |
| Plan → grocery list derivation and recalculation | FEAT-02, FEAT-03, FEAT-04, FEAT-05, FEAT-06, FEAT-23 | XBR-03 and XBR-04; every plan write (FEAT-02.SPEC-004, FEAT-03.SPEC-005, FEAT-04.SPEC-004, FEAT-23.SPEC-006) triggers FEAT-06.SPEC-002, which excludes pantry items (FEAT-05.SPEC-007) |
| Scheduled per-household background jobs (local-time aware) | FEAT-03, FEAT-04, FEAT-06, FEAT-07, FEAT-09, FEAT-13, FEAT-18, FEAT-21, FEAT-25 | Weekly generation (FEAT-03.SPEC-003), suggestion lapse (FEAT-04.SPEC-005), week rollover (FEAT-06.SPEC-004), plan-ready dispatch (FEAT-07.SPEC-001), invitation expiry (FEAT-09.SPEC-006), daily nudge (FEAT-13.SPEC-001), export and deletion cascades (FEAT-18.SPEC-006, FEAT-18.SPEC-008), calendar retries (FEAT-21.SPEC-004), check-in cycle (FEAT-25.SPEC-003) |
| Device-notification (push) delivery | FEAT-04, FEAT-07, FEAT-13, FEAT-23 | FEAT-07.SPEC-005 is the shared boundary used by FEAT-13.SPEC-002/004 and by FEAT-04.SPEC-006 and FEAT-23 pick suggestions (feature-dependency-map.md External Touchpoints, ASMP-31) |
| Transactional email delivery with queued retry | FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18 | Five Integration specs share ASMP-32 and the same "no user-visible effect, queued until recovery" degradation pattern (FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-14.SPEC-012, FEAT-18.SPEC-012 ## Degradation Behavior) |
| In-app notification inbox | FEAT-01, FEAT-02, FEAT-04, FEAT-09, FEAT-14, FEAT-18, FEAT-22, FEAT-24 | In-app channel in FEAT-01.SPEC-018, FEAT-02.SPEC-011/012/014, FEAT-04.SPEC-006, FEAT-09.SPEC-012..014, FEAT-14.SPEC-010/011, FEAT-18.SPEC-013, FEAT-22.SPEC-009/010, FEAT-24.SPEC-007 ## Channels |
| Tier gating from subscription state | FEAT-03, FEAT-05, FEAT-12, FEAT-14, FEAT-23 | XBR-05; FEAT-03.SPEC-009, FEAT-05.SPEC-005, FEAT-12.SPEC-004 all read the Subscription tier that FEAT-14.SPEC-008 writes |
| Role-based authorization including kid and operator roles | FEAT-01, FEAT-06, FEAT-09, FEAT-12, FEAT-14, FEAT-17, FEAT-18, FEAT-22 | FEAT-01.SPEC-016, FEAT-06.SPEC-009, FEAT-09.SPEC-011, FEAT-12.SPEC-003, FEAT-14.SPEC-006, FEAT-17.SPEC-006, FEAT-18.SPEC-011, FEAT-22.SPEC-006 Authorization Rules |
| Children's-privacy-class data handling across vendors | FEAT-01, FEAT-02, FEAT-03, FEAT-12, FEAT-17, FEAT-18, FEAT-22 | Data Sensitivity lines for Member Profile, Dietary Rule, Rating, Dinner Vote and Support Request (feature-dependency-map.md); dietary constraints leave the product in generation requests (FEAT-03.SPEC-010 ## Data Exchanged); export concentrates kid data (FEAT-18 ## Non-Functional Notes) |
| Locale-aware display (units, currency, aisles) | FEAT-03, FEAT-06, FEAT-08, FEAT-14, FEAT-16, FEAT-23, FEAT-25 | XBR-11; FEAT-16.SPEC-004 conversion consumed by plan cost, recipe quantities, list aisles, billing amounts (FEAT-14 ## Non-Functional Notes) and check-in spend |
| Recipe corpus and ingredient data quality | FEAT-02, FEAT-08, FEAT-10 | FEAT-08.SPEC-004 and FEAT-10.SPEC-005 both feed Recipe ingredients that FEAT-02.SPEC-007's completeness policy and FEAT-02.SPEC-002's allergen matching consume (XBR-19) |

## 4. Key Technical Risks

| Risk | Features Affected | Driving Evidence | Possible Mitigation Directions |
|------|-------------------|------------------|--------------------------------|
| Ingredient-to-allergen matching misses an allergen in free-text or compound ingredients, breaking the zero-incident promise | FEAT-02, FEAT-03, FEAT-04, FEAT-10, FEAT-17, FEAT-19, FEAT-23 | FEAT-02.SPEC-002 ## Edge Cases (compound terms), ## Processing Logic step 6; imported free text (FEAT-10.SPEC-005); no allergen-taxonomy option in technology-landscape.md | Run the FEAT-02 spike; layer a content-vendor taxonomy under a curated override list; conservative expansion of ambiguous terms; a regression suite of labeled recipes gating every rule or taxonomy change |
| The fail-closed completeness bar excludes a large share of recipes, leaving plans partial or repetitive for allergy-heavy households | FEAT-02, FEAT-03, FEAT-08, FEAT-10 | FEAT-02.SPEC-007 ## Field Validation Rules (quantity+unit on every ingredient); FEAT-03.SPEC-010 ## Edge Cases (small pools) | Measure the exclusion rate in the FEAT-02/FEAT-10 spikes; prompt users on the review screen to fill missing quantities; a product decision on "to taste" ingredients |
| LLM generation fails to fill a safe, in-budget week often enough, or exceeds the under-a-minute or cost envelope | FEAT-03, FEAT-04, FEAT-12, FEAT-05 | FEAT-03.SPEC-010 ## Edge Cases, ## Degradation Behavior; ASMP-23; SC-16 | Run the FEAT-03 spike across the AI & Intelligent Behavior options; deterministic pre-filtering to shrink prompts; a deterministic fallback selector; batch or cached-prompt pricing for the scheduled run |
| Offline grocery-list reconciliation diverges across devices (client clock skew, recalculation races, mobile service-worker limits) | FEAT-06, FEAT-05, FEAT-04, FEAT-23 | FEAT-06.SPEC-008 ## Cross-Field Rules (event-time LWW), ## Edge Cases; ASMP-25 | Run the FEAT-06 spike; server-assigned sequencing or hybrid logical clocks; stable line identity across recalculation; the Real-time & Collaboration options differ in ordering guarantees, which is a selection criterion |
| The swap chain (AI call + safety + write + recalculation + propagation) exceeds the under-10-seconds target | FEAT-04, FEAT-06, FEAT-13 | FEAT-04 ## Non-Functional Notes; FEAT-04.SPEC-007 ## Degradation Behavior (8-second slow note) | Precomputed alternatives from the verified pool; lower-latency model tiers; optimistic UI with server confirmation |
| Web Push reach on iOS depends on home-screen install, so plan-ready and nudge metrics underperform | FEAT-07, FEAT-13, FEAT-04 | FEAT-07.SPEC-005 ## Scope and Non-Goals (web app, no native, SC-05); FEAT-07 ## Non-Functional Notes (80% within one minute) | An install prompt as part of onboarding; the email fallback already specified (FEAT-07.SPEC-006); track reach by platform via Analytics & Product Telemetry |
| Children's data flows to third-party processors (LLM, auth, realtime, analytics, observability, email) beyond the parent-controlled boundary | FEAT-01, FEAT-02, FEAT-03, FEAT-12, FEAT-17, FEAT-18, FEAT-22 | ASMP-26, ASMP-27; FEAT-03.SPEC-010 ## Data Exchanged; Data Sensitivity lines in feature-dependency-map.md | Vendor data-processing terms and zero-retention settings as a selection criterion; minimize fields sent; scrub kid data from telemetry and session replay; self-host options (Trigger.dev, PostHog, Better Auth) where they reduce exposure |
| Deletion within 30 days misses copies outside the primary database | FEAT-18, FEAT-14, FEAT-21, FEAT-03 | FEAT-18.SPEC-008 ## Processing Logic (entity list omits processor, push, email, analytics, backup and export-file copies); ASMP-27 | A deletion inventory per vendor; object-storage lifecycle expiry for export files; backup retention at or under 30 days |
| Scheduled bursts at default time slots hit provider rate limits or job-platform quotas | FEAT-03, FEAT-07, FEAT-13 | platform-parameters.md `plan-arrival-time-slots`, `nightly-nudge-send-time`; FEAT-03.SPEC-003 ## Trigger Definition | Staggered dispatch within a slot; provider rate-limit discovery; job-platform pricing at the projected monthly run count |
| Payment webhooks arrive duplicated or out of order, corrupting billing_state or grace timing | FEAT-14, FEAT-03, FEAT-05, FEAT-12 | FEAT-14.SPEC-009 ## Inbound Events; FEAT-14.SPEC-007; feature-dependency-map.md, Subscription **Contention:** | Idempotent event handling keyed on processor event IDs; state-machine guards; a billing-management layer (Chargebee, Recurly) or merchant of record (Paddle) as alternatives to custom dunning |
| Legal status of web recipe import blocks FEAT-10 at v1 | FEAT-10 | ASMP-36; BRIEF.md Open Questions | A product/legal decision before v1; manual entry (FEAT-10.SPEC-003) remains a functional fallback |
| UK grocery ordering partner unavailable | FEAT-20 | technology-landscape.md Online Grocery Ordering Integration (no Tesco API found) | The FEAT-20 spike; a US-first scope; aggregator evaluation |

## 5. Open Questions for the Build Team

| # | Question | Why It Matters | What Would Resolve It |
|---|----------|----------------|------------------------|
| 1 | Does the organiser's affirmative confirmation (FEAT-01.SPEC-007) meet "verifiable parental consent" for US and UK children's-privacy obligations (ASMP-27)? | If not, FEAT-01 needs a consent-verification capability the landscape does not cover, which changes its integration scope | Legal review of COPPA/UK Age Appropriate Design Code obligations; if verification is required, a landscape addition for consent-verification services |
| 2 | Which ingredient-to-allergen taxonomy source backs the safety engine, vendor-provided or curated? | Drives FEAT-02's Hard verdict and the Recipe & Food Content Data selection | The FEAT-02 spike's recall and exclusion-rate results |
| 3 | Which LLM provider and prompt structure meet the under-a-minute, cost-per-household and full-week success targets? | Determines FEAT-03's feasibility path and FEAT-04's alternatives path | The FEAT-03 spike (success rate, p95 latency, tokens per run, monthly cost at projected households) |
| 4 | Should ingredients without a quantity or unit ("to taste", "a pinch") be exempt from the fail-closed completeness bar? | Affects how many starter and imported recipes are ever plan-eligible (FEAT-02.SPEC-007, FEAT-08, FEAT-10) | A product decision informed by the FEAT-02/FEAT-10 spike exclusion rates |
| 5 | Do the recipe-content vendors' licenses permit storing and redisplaying recipes in the household library, and at what tier cost versus the sub-$100/month pre-revenue budget? | Determines which Recipe & Food Content Data options are viable for FEAT-08 | Vendor terms review for Spoonacular, Edamam and licensed-content partners |
| 6 | Is importing recipes from other websites legally acceptable, and in what form (full steps vs ingredients plus link)? | Gates FEAT-10 at v1 (ASMP-36) | A founder/legal decision |
| 7 | What reach does Web Push achieve for the household base on iOS, and is a home-screen-install prompt acceptable in onboarding? | Affects FEAT-07's 80%-within-one-minute metric and FEAT-13's value | Early telemetry by platform; a product decision on install prompting |
| 8 | Do offline list edits reconcile on client event time or server-assigned order? | Separates a manageable FEAT-06 build from a divergence-prone one | The FEAT-06 spike under clock skew |
| 9 | Merchant of record (tax handled) or processor plus self-built dunning, given US and UK sales and a three-month revenue target? | Shapes FEAT-14's integration effort and compliance surface | Founder decision on tax-handling ownership vs per-transaction cost |
| 10 | Which grocery partners, and which markets, are in scope for the Later-phase handoff? | FEAT-20 verdict depends on UK availability | The FEAT-20 partner-discovery spike |
| 11 | Which external copies (processor, push subscriptions, email logs, analytics, backups, export files) fall inside the 30-day deletion promise? | FEAT-18 compliance with ASMP-27 | A vendor-by-vendor deletion inventory once the stack is selected |
| 12 | What identity and consent model applies to the Later-phase older-kid limited login? | FEAT-17 depends on it; managed identity providers' terms for minors' accounts vary | Resolution of BRIEF.md's open question on kids' representation, plus provider terms review |
| 13 | Should volume-to-weight conversion (cups to grams for solids) be supported? | FEAT-16 specifies only fixed same-dimension factors; UK households may expect grams | A product decision; if yes, per-ingredient density data from the Recipe & Food Content Data source |


## The Decisions — Technical Architecture

Section 1 below repeats the technical profile — by design; it is the architecture document's own embedded evidence base.


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
