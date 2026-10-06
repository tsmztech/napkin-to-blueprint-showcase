# FEAT-01 — Household Setup & Member Profiles

This chapter covers FEAT-01, Household Setup & Member Profiles, a Core-tier feature. It contains 18 specifications carrying 180 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-01.SPEC-001 | Account Sign-Up & Sign-In | screen | 9 |
| FEAT-01.SPEC-002 | Password Recovery | screen | 7 |
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | screen | 8 |
| FEAT-01.SPEC-004 | Member List & Add Member | screen | 10 |
| FEAT-01.SPEC-005 | Member Profile Detail | screen | 10 |
| FEAT-01.SPEC-006 | Dietary Rules Editor | screen | 11 |
| FEAT-01.SPEC-007 | Parental Consent Confirmation | screen | 7 |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | screen | 9 |
| FEAT-01.SPEC-009 | Setup Complete & Next Steps | screen | 9 |
| FEAT-01.SPEC-010 | Household Settings Hub | screen | 18 |
| FEAT-01.SPEC-011 | Default Subscription Provisioning | automation | 6 |
| FEAT-01.SPEC-012 | Mid-Week Hard-Rule Change Trigger | automation | 8 |
| FEAT-01.SPEC-013 | Setup Draft Persistence & Offline Queuing | logic-rule | 9 |
| FEAT-01.SPEC-014 | Household & Member Field Validation Rules | logic-rule | 14 |
| FEAT-01.SPEC-015 | Dietary Rule Classification & Allergen Matching Rules | logic-rule | 13 |
| FEAT-01.SPEC-016 | Household Setup Authorization Rules | logic-rule | 12 |
| FEAT-01.SPEC-017 | Transactional Email Integration (Account & Recovery) | integration | 10 |
| FEAT-01.SPEC-018 | Mid-Week Rule Change Notification | notification | 10 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Household Setup & Member Profiles

## Summary

**Feature:** Household Setup & Member Profiles
**ID:** FEAT-01
**Description:** The organiser sets up the household once: who eats with them, each person's allergies and diet, the weekly food budget, and how much time there is on which nights. This is the foundation every other feature reads from.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** The brief states the household "is set up once" before anything else can happen (BRIEF.md, Vision). Without this, there is no household to plan for. MVP: nothing else in the product functions without it. Independent reviews report that single-profile planners make mixed-diet families "hit walls quickly" and that entering several allergies leaves very few options, so per-person rules in one household are the differentiator.

**Key Capabilities:**
- Create an account and sign in — Organiser signs up with an email address and a protected sign-in before creating the household, and can recover access if the sign-in is forgotten
- Create the household — Organiser names the household and becomes its first member
- Add member profiles — Organiser adds each person eating with them, including kid profiles with no login
- Set dietary rules per person — Organiser records allergies, religious rules, per-person vegetarian settings, and known dislikes; allergies are chosen from a standard allergen list, with the option to name a specific extra ingredient, so the safety check can match them reliably
- Confirm parental consent for kid profiles — When adding a kid profile, the organiser confirms they are the child's parent or guardian and sees exactly what is stored: a first name or nickname, an age band, and dietary rules
- Set the weekly budget — Organiser states a rough weekly food budget
- Set the weekly schedule — Organiser marks which nights are short on time (e.g., 30-minute weeknights)
- Edit setup later — Organiser revisits and changes any of the above at any time

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-01.SPEC-001 | Account Sign-Up & Sign-In | Screen | Maya, Sam | New organiser creates a protected account, or an existing adult signs back in |
| FEAT-01.SPEC-002 | Password Recovery | Screen | Maya, Sam | An adult who cannot sign in requests and completes a sign-in reset |
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | Screen | Maya | Organiser names the household, becomes its first member, and enters the guided setup flow |
| FEAT-01.SPEC-004 | Member List & Add Member | Screen | Maya, Sam | Organiser sees every member and starts adding an adult or kid profile; Sam views the list |
| FEAT-01.SPEC-005 | Member Profile Detail | Screen | Maya, Sam | Organiser creates or edits one member's basic details (name, type, age band, notification preferences); Sam views a kid profile's details |
| FEAT-01.SPEC-006 | Dietary Rules Editor | Screen | Maya, Sam | Organiser records or edits one member's allergies, religious rules, vegetarian setting, and dislikes, and sees each rule's change history; Sam views a kid profile's dietary rules |
| FEAT-01.SPEC-007 | Parental Consent Confirmation | Screen | Maya | Organiser confirms parent/guardian status and sees exactly what a kid profile stores before it is created |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | Screen | Maya | Organiser states the weekly food budget and marks which nights are short on time |
| FEAT-01.SPEC-009 | Setup Complete & Next Steps | Screen | Maya | Organiser sees setup is complete and chooses between manual planning (free) or upgrading (paid) |
| FEAT-01.SPEC-010 | Household Settings Hub | Screen | Maya, Sam | Organiser revisits and edits any setup area later; Sam views household facts and links out to his own preferences |
| FEAT-01.SPEC-011 | Default Subscription Provisioning | Automation | All | On household creation, a free-tier Subscription record is created automatically |
| FEAT-01.SPEC-012 | Mid-Week Hard-Rule Change Trigger | Automation | Maya | A new or tightened hard dietary rule triggers an immediate re-check of the current week's plan |
| FEAT-01.SPEC-013 | Setup Draft Persistence & Offline Queuing | Logic/Rule | Maya | Every setup screen saves drafts locally and resumes exactly where the organiser left off, online or offline |
| FEAT-01.SPEC-014 | Household & Member Field Validation Rules | Logic/Rule | Maya | Governs household name, member cap, budget amount, and kid-profile data-minimality validation across setup screens |
| FEAT-01.SPEC-015 | Dietary Rule Classification & Allergen Matching Rules | Logic/Rule | Maya | Governs allergen selection, hard-vs-soft strength, per-person vegetarian logic, and the explicit-confirmation gate before an allergy is removed |
| FEAT-01.SPEC-016 | Household Setup Authorization Rules | Logic/Rule | All | Governs who can view or change each part of setup, including kid-profile data restrictions and the unauthorized-visitor experience |
| FEAT-01.SPEC-017 | Transactional Email Integration (Account & Recovery) | Integration | Maya, Sam | Sends account-creation confirmation and sign-in recovery emails through the transactional email capability |
| FEAT-01.SPEC-018 | Mid-Week Rule Change Notification | Notification | Maya | Tells the organiser which plan meal was removed after a mid-week hard-rule change |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Create an account and sign in | FEAT-01.SPEC-001, FEAT-01.SPEC-002, FEAT-01.SPEC-017, FEAT-01.SPEC-016 | Sign-up/sign-in screen, recovery screen, the email capability that delivers verification and reset messages, and the authorization rules gating unauthenticated access | Phase 2 (Explicit) |
| Create the household | FEAT-01.SPEC-003, FEAT-01.SPEC-014, FEAT-01.SPEC-011 | Household Naming screen creates the household and organiser member; field validation governs the name; default Subscription is provisioned as a side effect | Phase 2 (Explicit) |
| Add member profiles | FEAT-01.SPEC-004, FEAT-01.SPEC-005, FEAT-01.SPEC-014 | List/add screen and profile detail screen; validation governs the 12-member cap and kid-profile data minimality | Phase 2 (Explicit) |
| Set dietary rules per person | FEAT-01.SPEC-006, FEAT-01.SPEC-015, FEAT-01.SPEC-012, FEAT-01.SPEC-018 | Dietary Rules Editor screen; allergen/strength classification rules; a hard-rule change triggers the mid-week re-check and its notification | Phase 2 (Explicit) |
| Confirm parental consent for kid profiles | FEAT-01.SPEC-007 | Dedicated confirmation step shown before any kid profile is saved | Phase 2 (Explicit) |
| Set the weekly budget | FEAT-01.SPEC-008, FEAT-01.SPEC-014 | Budget & Schedule screen; validation governs the positive-amount rule | Phase 2 (Explicit) |
| Set the weekly schedule | FEAT-01.SPEC-008 | Budget & Schedule screen | Phase 2 (Explicit) |
| Edit setup later | FEAT-01.SPEC-010, FEAT-01.SPEC-013 | Household Settings Hub re-opens every setup screen in edit mode; draft persistence lets partial edits resume | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a single Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-01.SPEC-011 | Default Subscription Provisioning | Phase 4 (Trigger-Response — default data generation) | Household creation must not leave a household with no Subscription record; the Shared Data Entities slice states Subscription "defaults to free" and is "Created by FEAT-01" |
| FEAT-01.SPEC-012 | Mid-Week Hard-Rule Change Trigger | Phase 4 (Trigger-Response, cross-feature effect) | The Primary Flows & Alternates field and XBR-02 both require an immediate re-check when a hard rule changes; this feature owns the rule data that fires the trigger even though FEAT-02 runs the check |
| FEAT-01.SPEC-013 | Setup Draft Persistence & Offline Queuing | Phase 6 (Negative/Failure Analysis — Offline/Degraded, lost-data prevention) | The States field's Offline-degraded expectation and the journey's Failure/Recovery Variant both require setup to resume exactly where it was left, which is a rule shared across every setup screen |
| FEAT-01.SPEC-014 | Household & Member Field Validation Rules | Phase 5 (Rule-Constraint Discovery) | Five or more validation rules (name length, member cap, budget positivity, kid-data minimality, required organiser) apply across two entities and multiple screens — crosses the standalone-spec threshold |
| FEAT-01.SPEC-015 | Dietary Rule Classification & Allergen Matching Rules | Phase 5 (Rule-Constraint Discovery) | Allergen selection, hard/soft strength, per-person vegetarian logic, and the allergy-removal confirmation gate are conditional rules shared by the Dietary Rules Editor and read by the safety engine — the product's hard safety promise |
| FEAT-01.SPEC-016 | Household Setup Authorization Rules | Phase 5 (Rule-Constraint Discovery — authorization) | Every screen in this feature behaves differently by role (Maya Full, Sam View, both Jordan rows None, Riley View only via FEAT-22, unauthorized visitors None); this cross-cutting rule set is shared, not duplicated per screen |
| FEAT-01.SPEC-017 | Transactional Email Integration (Account & Recovery) | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-32) names transactional email as required for "account sign-up and sign-in recovery," naming FEAT-01 explicitly |
| FEAT-01.SPEC-018 | Mid-Week Rule Change Notification | Phase 4 (Notification surfacing) | The Communications field states "the organiser is also told when a mid-week rule change removes a meal from the current plan" — a message with a defined audience and content, not a same-screen toast |
| FEAT-01.SPEC-009 | Setup Complete & Next Steps | Phase 2 (Explicit) | First Household Setup journey, step 6: the guided setup's terminal screen, surfacing the free-tier "pick this week's dinners" vs. paid-tier "upgrade" branch rather than sitting under any single Key Capability |

## Entity-Lifecycle Coverage Matrix

**Entity: Household**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-01.SPEC-003 | Organiser names the household (1–60 characters) and becomes its first member | Also creates the organiser's Member Profile |
| Read (single) | FEAT-01.SPEC-010 | Household Settings Hub displays current household facts | Only one household exists per account (scope-boundaries.md SC-03), so there is no "Read (list)" |
| Read (list) | N/A | Not applicable — one household per account in v1 (scope-boundaries.md SC-03) | -- |
| Update | FEAT-01.SPEC-008, FEAT-01.SPEC-010 | Budget and schedule updates on the Budget & Schedule screen; household name edits from the Settings Hub | Locale fields (unit_system, currency, aisle_names) are updated by FEAT-16, not this feature |
| Delete/Archive | N/A — owned by Account & Data Management (FEAT-18) | Household deletion, including the 30-day removal window, is FEAT-18's responsibility per the dependency map ("Deleted by FEAT-18") and XBR-16; Household Setup never deletes the household | Explicit non-goal, see Non-Goals |
| State Transition | N/A — owned by FEAT-18 | The Active → Closed/Deleted transition is set only by household deletion (FEAT-18) | -- |

**Entity: Member Profile**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-01.SPEC-004, FEAT-01.SPEC-005 | Organiser adds an adult or kid profile from the member list; kid creation routes through FEAT-01.SPEC-007 first | Also created by FEAT-09 on invitation acceptance (not this feature's concern) |
| Read (single) | FEAT-01.SPEC-005 | Member Profile Detail screen | -- |
| Read (list) | FEAT-01.SPEC-004 | Member List screen, showing all members up to the 12-member cap | -- |
| Update | FEAT-01.SPEC-005 | Organiser edits display name, member type details, age band, and per-member notification preferences | -- |
| Delete/Archive | N/A — owned by Account & Data Management (FEAT-18) and Household Invitations & Membership (FEAT-09) | Member removal (FEAT-18) and a member leaving (FEAT-09) cascade-delete the member's dietary rules and ratings per XBR-16; Household Setup creates and edits profiles but never removes one | Explicit non-goal, see Non-Goals |
| State Transition | N/A — owned by FEAT-09 and FEAT-18 | Invited/Left/Removed states are set by invitation and account-management flows; this feature only establishes the initial Active status at creation (inline in FEAT-01.SPEC-004/005) | -- |

**Entity: Dietary Rule**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-01.SPEC-006 | Organiser picks an allergen from the standard list (optionally naming an extra ingredient), a religious rule, a per-person vegetarian setting, or a dislike | Also created by FEAT-12 as a learned soft dislike (not this feature's concern) |
| Read (single/list) | FEAT-01.SPEC-006, FEAT-01.SPEC-005 | Dietary Rules Editor lists a member's rules with change history; Member Profile Detail shows a summary | -- |
| Update | FEAT-01.SPEC-006 | Organiser edits strength, allergen, or named ingredient; every change is recorded to change_history | -- |
| Delete/Archive | FEAT-01.SPEC-006, FEAT-01.SPEC-015 | Hard delete: a dislike can be removed directly; an allergy or religious rule requires the explicit confirmation gate in FEAT-01.SPEC-015 before removal. No restore path — a removed rule is gone, but its change_history entry is retained indefinitely as part of the household's audit trail (ASMP-24 history depth). No cascade beyond the rule itself; removal via member deletion is owned by FEAT-18 (XBR-16) | Retention: change_history rows survive rule deletion for safety-investigation purposes; this is an explicit design decision, not an omission |
| State Transition | N/A | The product definition gives no state machine beyond the hard/soft strength field, which is set and edited inline in FEAT-01.SPEC-006 | -- |

**Entity: Subscription (created here as a side effect only)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-01.SPEC-011 | Household creation automatically provisions a free-tier Subscription record | -- |
| Read / Update / Delete/Archive / State Transition | N/A — owned entirely by Subscription & Billing Management (FEAT-14) | All other Subscription lifecycle operations (upgrade, billing period, cancellation, grace period) belong to FEAT-14 | This feature only ever creates the default record |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Support Request | FEAT-01.SPEC-010 | Household Settings Hub shows the organiser each time support (Riley) viewed the household and why (access_record), per the Data Notes field and XBR-14 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Household record is created | A free-tier Subscription record is provisioned automatically | Standalone Automation | FEAT-01.SPEC-011 |
| A new or tightened hard dietary rule (allergy or religious rule) is saved | Current week's plan is re-checked; failing meals are flagged; safe alternatives and a grocery-list update follow through FEAT-02/FEAT-04/FEAT-06 | Standalone Automation | FEAT-01.SPEC-012 |
| The mid-week re-check removes a meal from the current plan | Organiser is told which meal was removed | Standalone Notification | FEAT-01.SPEC-018 |
| Organiser attempts to remove an already-entered allergy or religious rule | Removal is blocked until an explicit confirmation step is completed | Standalone Logic/Rule | FEAT-01.SPEC-015 |
| Organiser selects "kid profile" as member type | Parental consent confirmation is required before the profile can be saved | Inline in triggering screen (screen-to-screen flow already modeled) | FEAT-01.SPEC-007 |
| Organiser attempts to add a 13th member profile | Add action is blocked with a message citing the 12-member cap | Standalone Logic/Rule | FEAT-01.SPEC-014 |
| A setup step fails to save | Entered data stays on screen; a retry option is offered; nothing is silently discarded | Inline in triggering screen | FEAT-01.SPEC-003 / 004 / 005 / 006 / 008 |
| Organiser closes the app mid-setup, with or without connectivity | Draft is held locally; setup resumes exactly where it was left off when reopened | Standalone Logic/Rule | FEAT-01.SPEC-013 |
| Organiser completes setup | Confirmation shown, branching by subscription tier (manual planning vs. AI plan) | Inline in triggering screen | FEAT-01.SPEC-009 |
| New account is created | Account-confirmation email is sent through the transactional email capability | Standalone Integration | FEAT-01.SPEC-017 |
| Adult requests sign-in recovery | Password-reset email is sent through the transactional email capability | Standalone Integration | FEAT-01.SPEC-017 |
| Unauthorized visitor attempts to reach household setup | Visitor is shown only a sign-in or invitation-acceptance screen, never household data | Standalone Logic/Rule | FEAT-01.SPEC-016 |

## Shared Context

**Shared Entities:**
- Household — created by FEAT-01.SPEC-003, updated by FEAT-01.SPEC-008 and FEAT-01.SPEC-010, read by FEAT-01.SPEC-010. Fields: household_name, organiser, weekly_budget, weekly_schedule, unit_system, currency, aisle_names, plan_arrival_day_time, status.
- Member Profile — created by FEAT-01.SPEC-004/005, read/listed by FEAT-01.SPEC-004/005, updated by FEAT-01.SPEC-005. Fields: display_name, member_type, sign_in, age_band, parental_consent_confirmation, notification_preferences, status.
- Dietary Rule — created, read, and updated by FEAT-01.SPEC-006; deletion gated by FEAT-01.SPEC-015. Fields: member, rule_kind, strength, allergen, origin, change_history.
- Subscription — created only, by FEAT-01.SPEC-011, as a side effect of Household creation. No fields of this feature's concern beyond the free-tier default.

**Shared UI Patterns:**
- Guided setup wizard shell — FEAT-01.SPEC-003, 004, 005, 006, 007, 008, and 009 share one step-by-step wizard chrome (progress indication, inline save confirmation instead of a full-page spinner, back/continue navigation). Spec Writers should describe this shell consistently across all seven screens rather than redefining it per screen.
- Member profile card — the same summary card (name, type, dietary-rule badges) appears on FEAT-01.SPEC-004's list and is the entry point into FEAT-01.SPEC-005/006 for that member.
- Confirmation-required destructive action — the allergy-removal confirmation (FEAT-01.SPEC-015) and the parental-consent confirmation (FEAT-01.SPEC-007) share the same modal pattern: state what will happen, require an explicit affirmative tap, no default-confirmed state.

**Shared Validation:**
- FEAT-01.SPEC-014 defines household- and member-level field validation (name length, member cap, budget positivity, kid-data minimality). FEAT-01.SPEC-003, 004, 005, and 008 all reference SPEC-014 rather than duplicating these rules.
- FEAT-01.SPEC-015 defines dietary-rule classification and the allergy-removal confirmation gate. FEAT-01.SPEC-006 references SPEC-015 for all allergen, strength, and removal behavior.
- FEAT-01.SPEC-016 defines who may view or change each part of setup. Every screen in this feature (FEAT-01.SPEC-001 through 010) references SPEC-016 for its role-gated behavior rather than restating access rules per screen.

## Internal Dependency Map

```
SPEC-001 (Account Sign-Up & Sign-In) -> [new account created] -> SPEC-017 (Transactional Email Integration) -> [verification email sent] -> SPEC-003 (Household Naming & Guided Setup Start)
SPEC-001 (Account Sign-Up & Sign-In) -> [existing organiser signs in, household exists] -> SPEC-010 (Household Settings Hub)
SPEC-001 (Account Sign-Up & Sign-In) -> [taps "forgot password"] -> SPEC-002 (Password Recovery) -> [reset requested] -> SPEC-017 (Transactional Email Integration)
SPEC-003 (Household Naming & Guided Setup Start) -> [household created] -> SPEC-011 (Default Subscription Provisioning)
SPEC-003 (Household Naming & Guided Setup Start) -> [organiser continues] -> SPEC-004 (Member List & Add Member)
SPEC-004 (Member List & Add Member) -> [taps "add adult" or "add kid"] -> SPEC-005 (Member Profile Detail)
SPEC-005 (Member Profile Detail) -> [kid profile type selected] -> SPEC-007 (Parental Consent Confirmation) -> [confirmed] -> SPEC-005 (Member Profile Detail)
SPEC-005 (Member Profile Detail) -> [taps "dietary rules"] -> SPEC-006 (Dietary Rules Editor)
SPEC-006 (Dietary Rules Editor) -> [validates allergen and strength using] -> SPEC-015 (Dietary Rule Classification & Allergen Matching Rules)
SPEC-006 (Dietary Rules Editor) -> [hard rule added or tightened] -> SPEC-012 (Mid-Week Hard-Rule Change Trigger) -> SPEC-018 (Mid-Week Rule Change Notification)
SPEC-004 (Member List & Add Member) -> [all members added, continues] -> SPEC-008 (Weekly Budget & Schedule Setup)
SPEC-008 (Weekly Budget & Schedule Setup) -> [continues] -> SPEC-009 (Setup Complete & Next Steps)
SPEC-010 (Household Settings Hub) -> [taps any setup area] -> SPEC-003 / SPEC-004 / SPEC-005 / SPEC-006 / SPEC-008 (in edit mode)
SPEC-003 / SPEC-004 / SPEC-005 / SPEC-006 / SPEC-008 -> [validates fields using] -> SPEC-014 (Household & Member Field Validation Rules)
SPEC-003 / SPEC-004 / SPEC-005 / SPEC-006 / SPEC-008 -> [navigation away or connectivity lost] -> SPEC-013 (Setup Draft Persistence & Offline Queuing) -> [reopened] -> same screen, resumed
SPEC-001 through SPEC-010 -> [gated by] -> SPEC-016 (Household Setup Authorization Rules)
```

**Default Entry:** SPEC-001 (Account Sign-Up & Sign-In) for a first-time visitor with no account; SPEC-010 (Household Settings Hub) for a returning organiser whose household already exists.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-01.SPEC-003 | Outbound | FEAT-16 (Units, Currency & Locale Configuration) | Guided setup hands off to units, currency, and aisle-layout settings | Setup step "Set units, currency and aisle layout" (First Household Setup journey, step 4) |
| FEAT-01.SPEC-004 | Outbound | FEAT-09 (Household Invitations & Membership) | Organiser sends a partner an invitation from within setup | Tap "invite" during setup (First Household Setup journey, step 5) |
| FEAT-01.SPEC-009 | Outbound | FEAT-23 (Manual Weekly Planning) | Setup completion hands off to picking this week's dinners on the free tier | Choose "pick this week's dinners" (First Household Setup journey, step 6) |
| FEAT-01.SPEC-009 | Outbound | FEAT-14 (Subscription & Billing Management) | Setup completion hands off to the paid-tier overview | Choose "upgrade" (First Household Setup journey, step 6) |
| FEAT-01.SPEC-010 | Outbound | FEAT-07 | Household settings exposes the plan-ready notification preference and plan-arrival day | Organiser opens notification settings from the Settings Hub |
| FEAT-01.SPEC-010 | Outbound | FEAT-13 | Household settings exposes the nightly-nudge preference | Organiser opens notification settings from the Settings Hub |
| FEAT-01.SPEC-010 | Outbound | FEAT-22 (Operator Read-Only Support Access) | Organiser opens the record of when and why support viewed the household | Organiser opens "support access" from the Settings Hub |
| FEAT-01.SPEC-010 | Outbound | FEAT-21 | Household settings exposes a calendar connection entry point (Later phase) | Organiser taps "connect calendar" |
| FEAT-01.SPEC-012 | Outbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Hard-rule change hands off the actual re-check of the current week's plan | A hard dietary rule is added or tightened |
| FEAT-01.SPEC-012 | Outbound | FEAT-04 (One-Tap Meal Swap) | Safe alternatives are offered for any meal the re-check removes | FEAT-02 flags a now-unsafe meal |
| FEAT-01.SPEC-006 | Outbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Every dietary rule recorded here is read by the safety engine against candidate recipe ingredients | Rule created or edited |
| FEAT-01.SPEC-004 | Inbound | FEAT-09 (Household Invitations & Membership) | An accepted invitation adds a new Member Profile to the list this feature manages | Invited partner accepts |
| FEAT-01.SPEC-003 | Inbound | FEAT-24 | A new household is started from a referral welcome page | Referred visitor follows the link and starts setup (End-of-Week Check-In journey, step 4) |

## Non-Functional Notes

**Data volumes / growth:** A household holds up to 12 member profiles, comfortably above the brief's 2–6 people (FEAT-01 Validation & Limits); the product expects several thousand such households in the first year, and every household's setup history — including each dietary rule's change history — is kept for the life of the account (assumptions-constraints.md ASMP-24; scope-boundaries.md SC-15, SC-18).

**Responsiveness:** Saving a setup step shows an inline confirmation rather than a full-page spinner, and a failed save keeps the entered data on screen with a retry option (FEAT-01 States field); the heavier responsiveness targets in assumptions-constraints.md ASMP-23 (weekly plan generation, recipe search) do not apply to this feature's setup screens.

**Data sensitivity / privacy:** Dietary Rule data, including children's allergy information, is the most sensitive data in the product — children's-privacy-class protection, parent-controlled, used only for the household's own plan, never sold or used for advertising (assumptions-constraints.md ASMP-26, ASMP-27). A kid profile stores only a first name or nickname, an age band, and dietary rules — no surname, birth date, photo, or contact detail (FEAT-01 Validation & Limits). Each dietary rule's change history and every support-access visit are visible to the organiser for trust and safety-investigation purposes (FEAT-01 Data Notes).

**Compliance flags:** Because the product processes children's data, kid-profile creation requires the organiser's verifiable parental consent (FEAT-01.SPEC-007) and the data collected is held to children's-privacy-class minimal-collection and parental-control standards; general personal-data rights (export, deletion) for all household members are delivered through Account & Data Management (FEAT-18), not this feature (assumptions-constraints.md ASMP-27).

## Non-Goals

- **Deleting the household or removing a member** — Owned by Account & Data Management (FEAT-18) per the dependency map and XBR-16; Household Setup & Member Profiles creates and edits household and member data but never deletes it. Excluding deletion here keeps a single feature accountable for the 30-day removal window and its cascade behavior.
- **Multiple households per account** — Excluded per scope-boundaries.md SC-03: BRIEF.md states plainly "there is one household per account in v1," so this feature never offers a household switcher or a second household.
- **Independent kid sign-in in v1** — Excluded per scope-boundaries.md SC-02: the founder's stated v1 default is parent-managed profiles with no login for young kids; any kid login is the distinct, Later-phase FEAT-17, not part of this feature's guided setup.
- **Other adult members changing household setup or approving the plan** — Excluded per scope-boundaries.md SC-04: BRIEF.md gives setup, dietary rules, budget, schedule, and plan approval to the organiser alone; Sam's View access lets him see but never change these facts, and the organiser role changes hands only through FEAT-09's hand-over.
- **Medical or diet advice derived from dietary rules** — Excluded per scope-boundaries.md SC-06: BRIEF.md states directly "there is no medical or diet advice"; the Dietary Rules Editor records allergies, religious rules, and preferences for matching purposes only, never as nutritional guidance.
- **Organiser hand-over** — Owned by Household Invitations & Membership (FEAT-09) per XBR-15: this feature requires that a household always have exactly one organiser, but the hand-over flow itself (recipient acceptance, the guarantee against a household ever having zero organisers) is FEAT-09's responsibility, not a Household Setup screen.



# Screen Spec: Account Sign-Up & Sign-In

## Overview

**Name:** Account Sign-Up & Sign-In
**ID:** FEAT-01.SPEC-001
**Type:** Screen
**Purpose:** A new organiser creates a protected account with an email address and sign-in, or an existing adult household member signs back in.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Creating a new account (email address plus a protected sign-in) for a first-time organiser
- Signing in an existing adult member (Maya or Sam) with an established account
- Routing a brand-new account into household creation, and an existing organiser's account into the household they already have
- Entry point into Password Recovery (FEAT-01.SPEC-002) for an adult who cannot sign in

**Non-Goals:**
- Accepting a household invitation -- an invited adult's account creation and joining flow belongs to Household Invitations & Membership (FEAT-09); this screen only handles a person creating or accessing their own account directly
- Any kid-profile login -- excluded per scope-boundaries.md SC-02: young kid profiles have no login in v1, and the Later-phase older-kid limited login (FEAT-17) is a distinct mechanism, not delivered by this screen
- Operator (Riley) authentication -- support access is delivered entirely through the separate read-only support view (FEAT-22, XBR-14); this screen never authenticates an operator

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External / default entry | First-time visitor opens the product with no active session | None -- screen starts in its default "sign up or sign in" state |
| FEAT-24 (Invite Another Household), referral welcome page | Referred visitor follows a household referral link and chooses to start | Referring household's member first name, shown as a welcome context; no other household data |
| FEAT-01.SPEC-002 (Password Recovery) | Reset completes successfully | Message confirming the sign-in was reset; email address pre-filled |
| Any screen, expired session | Session expires while the user is elsewhere in the product | The screen the user was on, to return to after re-authenticating |
| FEAT-09.SPEC-002 (Invitation Acceptance) | Invitee taps "Create your own household instead" | None -- screen starts in its default "sign up or sign in" state |
| FEAT-09.SPEC-005 (Leave Household) | Other Adult Member completes leaving the household | None -- the member no longer belongs to a household |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Create a new account, or sign in to an existing one | -- |
| Sam (Other Adult Member) | Full screen | Sign in to an existing account (created when accepting an invitation via FEAT-09) | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- young kid profiles have no login; Jordan's data is managed entirely through Maya's account (FEAT-01.SPEC-005, FEAT-01.SPEC-006) |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- the Later-phase older-kid login is a distinct mechanism (FEAT-17), not delivered by this screen |
| Riley (Operator, support) | No | No | N/A -- operator support access is delivered only through the separate read-only support view (FEAT-22); this screen has no operator path |
| Unauthenticated | Yes | Yes (sign up or sign in) | This is the default destination for an unauthenticated visitor -- no restriction to describe |
| Expired session | Yes | Yes | Dialog: "Your session has expired. Sign in to continue." The screen the user was on is remembered and reopened automatically after a successful sign-in; any unsaved setup draft is preserved by FEAT-01.SPEC-013 |

single-role restriction note: this screen serves both adult roles identically -- there is no role-specific behavior on the screen itself; role differences begin only after authentication, on the screens the user is routed to next.

## Layout and Content

**Header:** Product name/logo, centered. No back navigation (this is the entry screen).

**Body:** A single form area with two modes, switched by a text toggle above the fields:
- **Sign up** (default for a first-time visitor): Email address field, Password field (with a visible strength indicator), a "Create account" button.
- **Sign in** (default when arriving from a referral link's "I already have an account" choice, or when a returning visitor's device previously signed in): Email address field, Password field, a "Forgot password?" link, a "Sign in" button.

Below the form, a text toggle: "Already have an account? Sign in" (in sign-up mode) or "New here? Create an account" (in sign-in mode), switching the mode without leaving the screen.

When arrived from a referral link (FEAT-24), a single line appears above the form: "{referring member first name} invited you to try Plateful" -- no other referring-household data is shown.

**Footer:** Legal text line linking to terms and privacy information (static content, not interactive beyond the links themselves).

### Responsive Behavior

- **Compact breakpoint:** Form fields stack full width; the mode toggle and footer remain visible below the fold with the form scrollable above them.
- **Medium size class and above:** Form area caps at a consistent platform-wide narrow width and is horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Mode toggle ("Sign in" / "Create an account") | Tap | Switches the form between sign-up and sign-in field sets | Form fields and primary button label change | Immediate visual swap, no loading state |
| Email address field | Type | Captures the email address | Field shows entered text | Standard input focus state |
| Email address field | Blur | Validates format via FEAT-01.SPEC-014 | Error state if invalid | "Enter a valid email address" below the field |
| Password field (sign up) | Type | Captures the password; strength indicator updates live | Indicator reflects strength (weak/adequate/strong) | Live indicator update, no blocking feedback while typing |
| Password field (sign up) | Blur | Validates minimum protection requirement | Error state if requirement not met | "Choose a password that is harder to guess" below the field |
| "Forgot password?" link (sign-in mode only) | Tap | Navigate to FEAT-01.SPEC-002 (Password Recovery) | Screen changes | Standard navigation transition |
| "Create account" button (sign-up mode) | Tap | 1. Validate email and password. 2. If valid, create the account. 3. Route to FEAT-01.SPEC-003 (new account, no household yet). | Button shows loading state | Success: navigates directly to household naming. Failure: inline error (e.g., "An account with this email already exists -- sign in instead") with a one-tap switch to sign-in mode pre-filled with the entered email |
| "Sign in" button (sign-in mode) | Tap | 1. Validate credentials against the stored account. 2. Route to FEAT-01.SPEC-010 (organiser with an existing household) or FEAT-01.SPEC-003 (an account somehow without a household, e.g. interrupted first setup) based on account state. | Button shows loading state | Success: navigates to the screen determined by step 2 (Settings Hub or household naming). Failure: "That email and password don't match. Try again or reset your password." with the "Forgot password?" link emphasized |
| "Create account" / "Sign in" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Mode toggle -> Email address -> Password -> (Forgot password link, sign-in mode only) -> primary submit button.
- **Validation announcements:** Field errors are announced to assistive technology and programmatically associated with their field when they appear.
- **Mode switch announcement:** Switching between sign-up and sign-in is announced ("Now showing: sign in") so the field-set change is not silently missed by screen reader users.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Sign-up (default, new visitor) | Sign-up fields, "Create account" button | Screen opens with no prior session on this device | User switches mode, submits, or navigates away |
| Sign-in (returning visitor) | Sign-in fields, "Sign in" button | Screen opens on a device that previously signed in, or user switches mode | User switches mode, submits, or navigates away |
| Submitting | Primary button shows loading state, fields disabled | User taps Create account or Sign in with valid input | Submission succeeds or fails |
| Error | Inline error message shown; fields remain editable and retain entered values | Submission fails (validation, existing account, or credential mismatch) | User corrects input and resubmits |
| Offline/Degraded | Banner: "You're offline. Sign-up and sign-in need a connection -- try again once you're back online." Fields remain visible and editable but the primary button is disabled | Connectivity is lost while this screen is open | Connectivity returns -- banner clears and the primary button re-enables |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules) for account-level fields. This screen applies validation on field blur and on form submission.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful account creation | FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | -- |
| Successful sign-in, household already exists | FEAT-01.SPEC-010 (Household Settings Hub) | -- |
| Successful sign-in, no household yet | FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | -- |
| "Forgot password?" tap | FEAT-01.SPEC-002 (Password Recovery) | -- |

## Data Model

**Creates:** Member Profile (organiser) -- display_name is captured later at household naming (FEAT-01.SPEC-003); at account creation only the sign_in credential (email and protected sign-in) is established for the account holder.
**Reads:** None on entry; on sign-in, the account's stored sign_in credential is checked against the entered values.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Account creation triggers FEAT-01.SPEC-017 (Transactional Email Integration), which sends an account-confirmation email.
- A newly created account with no household proceeds directly to FEAT-01.SPEC-003 -- there is no separate "verify your email before continuing" gate blocking setup, per the product's guided-setup-first flow (BRIEF.md, Vision).
- An account may hold only one household in v1 (scope-boundaries.md SC-03); sign-in routes an existing organiser straight to their one household's Settings Hub rather than any household picker.
- Authorization for what a signed-in account can subsequently see and do is governed by FEAT-01.SPEC-016 (Household Setup Authorization Rules).

## Edge Cases

- **User attempts to create an account with an email already in use** -- Inline error: "An account with this email already exists -- sign in instead," with a one-tap switch to sign-in mode, email pre-filled.
- **User taps "Create account" / "Sign in" twice rapidly** -- Second tap is ignored while the first submission is in progress (button in loading state).
- **User navigates away mid-entry** -- No confirmation dialog; this screen holds no destructive unsaved state (account creation has not started), so the entered email/password are simply discarded.
- **Referral link visitor already has an account** -- Signing in proceeds normally; per XBR-20, a person who already has a household is told so and no new referral is recorded.
- **Concurrent sign-in from two devices** -- Not a conflict: nothing on this screen is shared, mutable state; each device establishes its own session independently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-002 (Password Recovery) | Navigation (outbound) | "Forgot password?" leads here |
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Navigation (outbound) | New or household-less accounts land here |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (outbound) | Returning organisers with an existing household land here |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Email format and password rules |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Post-sign-in authorization |
| FEAT-01.SPEC-017 (Transactional Email Integration) | Triggers (outbound) | Sends the account-confirmation email |
| FEAT-24 (Invite Another Household) | Navigation (inbound) | Referral welcome page routes here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| account_created | entry source (default / referral) | New account is successfully created | supports success-metrics.md: "First-Session Onboarding Completion" |
| account_sign_in_succeeded | account age in days | An existing account signs in successfully | N/A -- no Stage 2 metric measures return sign-ins directly; retained to distinguish new-account starts from returning sessions when reading onboarding funnel data |
| account_sign_in_failed | failure reason (credential mismatch) | A sign-in attempt fails | N/A -- diagnostic signal only, no Stage 2 metric measures sign-in failure rate |

## Acceptance Criteria

**FEAT-01.SPEC-001-AC-01:** Given Maya is a first-time visitor on this screen, when she enters a valid email and a password that meets the strength requirement and taps "Create account", then her account is created and she is taken to FEAT-01.SPEC-003 (Household Naming & Guided Setup Start).

**FEAT-01.SPEC-001-AC-02:** Given Maya's account already has a household, when she signs in with the correct email and password, then she is taken directly to FEAT-01.SPEC-010 (Household Settings Hub).

**FEAT-01.SPEC-001-AC-03:** Given Sam has an existing account created when he accepted Maya's invitation, when he signs in with his correct credentials, then he is taken to FEAT-01.SPEC-010 (Household Settings Hub) with his own (View) access.

**FEAT-01.SPEC-001-AC-04:** Given a visitor attempts to create an account with an email already registered, when they tap "Create account", then the error "An account with this email already exists -- sign in instead" appears with a one-tap switch to sign-in mode.

**FEAT-01.SPEC-001-AC-05:** Given a visitor enters an email and password that do not match any stored account, when they tap "Sign in", then the error "That email and password don't match. Try again or reset your password." appears.

**FEAT-01.SPEC-001-AC-06:** Given a visitor is on the sign-in form, when they tap "Forgot password?", then they are taken to FEAT-01.SPEC-002 (Password Recovery).

**FEAT-01.SPEC-001-AC-07:** Given a visitor arrived from a household referral link, when the screen loads, then the line "{referring member first name} invited you to try Plateful" appears above the form and no other household data from the referring household is shown.

**FEAT-01.SPEC-001-AC-08:** Given a user's session expires while they were mid-setup, when they are redirected to this screen, then the dialog "Your session has expired. Sign in to continue." appears, and after successful sign-in they are returned to the screen and draft they were on.

**FEAT-01.SPEC-001-AC-09:** Given Maya loses connectivity on this screen, when she attempts to submit, then the banner "You're offline. Sign-up and sign-in need a connection -- try again once you're back online." appears and the primary button is disabled until connectivity returns.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 5 (sign-up, sign-in, submitting, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Password Recovery

## Overview

**Name:** Password Recovery
**ID:** FEAT-01.SPEC-002
**Type:** Screen
**Purpose:** An adult who cannot sign in requests a sign-in reset by email and completes it.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Requesting a sign-in reset for a given email address
- Confirming the reset request was sent, without revealing whether the email is registered
- Completing the reset from the emailed link: setting a new password and returning to sign-in

**Non-Goals:**
- Sending the reset email itself -- delivered through FEAT-01.SPEC-017 (Transactional Email Integration); this screen only requests and completes the reset
- Account recovery for a forgotten email address -- product-features.md defines no email-recovery path; an adult who has lost access to both their password and their registered email must contact support (FEAT-18, general support contact)
- Kid-profile or operator sign-in recovery -- excluded per scope-boundaries.md SC-02 and this feature's Non-Goals: young kid profiles have no login to recover, and operator access is never authenticated through this consumer flow

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | User taps "Forgot password?" | Email address, if already entered on the sign-in form |
| Reset email link | User taps the reset link in the emailed message | A single-use reset token identifying the account |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Request and complete a sign-in reset for her own account | -- |
| Sam (Other Adult Member) | Full screen | Request and complete a sign-in reset for his own account | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- young kid profiles have no login to recover |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- the Later-phase older-kid login's recovery mechanism is out of this feature's scope |
| Riley (Operator, support) | No | No | N/A -- operator access is never authenticated through this consumer recovery flow |
| Unauthenticated | Yes | Yes (request and complete a reset) | This screen is reachable without an active session by design -- no restriction to describe |
| Expired session | Yes | Yes | Treated identically to unauthenticated: a reset can be requested or completed without a live session |

single-role restriction note: this screen serves both adult roles identically -- there is no role-specific behavior, since a password reset always acts on the requesting person's own account only.

## Layout and Content

**Header:** "Reset your sign-in" title with a back arrow returning to FEAT-01.SPEC-001.

**Body -- Request step:** Single field, "Email address," and a "Send reset link" button below it.

**Body -- Confirmation step (after request submitted):** A single message: "If an account exists for {entered email}, a reset link is on its way. Check your inbox." No other content or action beyond a "Back to sign in" link.

**Body -- Reset step (arrived via emailed link):** "New password" field with a strength indicator (same treatment as sign-up), a "Confirm new password" field, and a "Set new password" button.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single field or fields stack full width; button full width below.
- **Medium size class and above:** Content area caps at the same narrow platform-wide form width as FEAT-01.SPEC-001, horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Screen closes | Standard transition back to sign-in |
| Email address field (request step) | Type | Captures the email address | Field shows entered text | Standard input focus state |
| "Send reset link" button | Tap | 1. Validate email format. 2. Trigger FEAT-01.SPEC-017 to send a reset email if an account exists for that address. | Button shows loading state | Always transitions to the confirmation message, regardless of whether the email is registered (no account-existence disclosure) |
| "Back to sign in" link (confirmation step) | Tap | Navigate to FEAT-01.SPEC-001 | Screen closes | Standard transition |
| New password field (reset step) | Type | Captures the new password; strength indicator updates live | Indicator reflects strength | Live update |
| Confirm new password field (reset step) | Blur | Compares against the new password field via FEAT-01.SPEC-014 | Error state if mismatched | "Passwords don't match" below the field |
| "Set new password" button (reset step) | Tap | 1. Validate both password fields. 2. Apply the new sign-in credential to the account. 3. Sign the user in. | Button shows loading state | Success: navigates to FEAT-01.SPEC-010 (or FEAT-01.SPEC-003 if the account has no household yet) with a confirmation toast "Sign-in updated." Failure: inline error, reset link intact for retry |

### Accessibility Notes

- **Focus order:** Back arrow -> Email address (request step) or New password -> Confirm new password -> Set new password (reset step).
- **Validation announcements:** Password-mismatch and format errors are announced and associated with their field.
- **Confirmation announcement:** The "reset link is on its way" message is announced on the confirmation step so a screen reader user is not left waiting silently.
- **Keyboard alternatives:** Every action is keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Request (default) | Email field and "Send reset link" button | Arrived from "Forgot password?" | User submits the request |
| Confirmation | Neutral confirmation message, no form fields | Reset request submitted (regardless of outcome) | User taps "Back to sign in" or navigates away |
| Reset (link opened) | New password and confirm fields, "Set new password" button | Arrived via a valid, unexpired reset link | User submits a valid new password |
| Reset link invalid or expired | Message: "This reset link is no longer valid. Request a new one." with a "Request new link" button returning to the Request state | Reset token is expired, already used, or malformed | User requests a new link |
| Error | Inline error message; fields retain entered values | Reset submission fails validation or the credential update fails | User corrects input and resubmits |
| Offline/Degraded | Banner: "You're offline. Password reset needs a connection -- try again once you're back online." | Connectivity lost while this screen is open | Connectivity returns |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules) for email format and new-password strength/match rules. Checked on blur and on submission.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | -- |
| "Back to sign in" link tap | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | -- |
| Successful reset, household exists | FEAT-01.SPEC-010 (Household Settings Hub) | -- |
| Successful reset, no household yet | FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | -- |
| "Request new link" tap (expired link) | Request state of this same screen | -- |

## Data Model

**Creates:** None.
**Reads:** None displayed; the reset token is validated against the account it was issued for.
**Updates:** The account's sign_in credential, on successful completion of the reset step.
**Deletes:** None.

## Business Rules

- Requesting a reset never discloses whether the entered email is registered -- the confirmation message is identical either way, protecting household members' privacy.
- A reset link is single-use and time-limited; using it or letting it expire invalidates it (see FEAT-01.SPEC-017 for delivery and FEAT-01.SPEC-014 for the token's validity window).
- Completing a reset signs the user in immediately -- no separate sign-in step is required afterward.
- Sending the reset email is delegated entirely to FEAT-01.SPEC-017 (Transactional Email Integration); this screen only triggers the request and consumes the resulting link.

## Edge Cases

- **User requests a reset for an unregistered email** -- The confirmation message still appears (no disclosure); no email is actually sent.
- **User taps the reset link twice (opens it in two tabs)** -- The first completed reset invalidates the token; the second tab shows "This reset link is no longer valid. Request a new one." if submitted after the first succeeds.
- **User requests a second reset before completing the first** -- The newer link becomes the valid one; the earlier link is invalidated to prevent confusion about which is current.
- **User submits mismatched new passwords** -- Inline error "Passwords don't match"; the reset link remains valid for further attempts until it expires.
- **User taps "Send reset link" twice rapidly** -- Second tap is ignored while the first request is in flight.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (inbound/outbound) | Entry via "Forgot password?"; returns here on cancel |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (outbound) | Successful reset for an existing household lands here |
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Navigation (outbound) | Successful reset for a household-less account lands here |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Password strength/match and email format rules |
| FEAT-01.SPEC-017 (Transactional Email Integration) | Triggers (outbound) | Sends the reset-link email |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| password_reset_requested | -- (no email captured in the event to avoid logging account-existence signal) | User submits the request step | N/A -- no Stage 2 metric tracks recovery volume; retained as an operational signal for support load |
| password_reset_completed | time since request | User successfully sets a new password | supports success-metrics.md: "First-Session Onboarding Completion" (a blocked sign-in resolved quickly keeps a returning organiser's session productive rather than abandoned) |

## Acceptance Criteria

**FEAT-01.SPEC-002-AC-01:** Given Maya cannot remember her password, when she taps "Forgot password?" on FEAT-01.SPEC-001 and enters her registered email and taps "Send reset link", then she sees "If an account exists for {email}, a reset link is on its way. Check your inbox."

**FEAT-01.SPEC-002-AC-02:** Given Sam enters an email that is not registered and taps "Send reset link", then he sees the identical confirmation message as a registered email would produce, and no reset email is sent.

**FEAT-01.SPEC-002-AC-03:** Given Maya opens a valid, unexpired reset link, when she enters a new password meeting the strength requirement in both fields and taps "Set new password", then her sign-in credential updates and she is signed in and taken to FEAT-01.SPEC-010.

**FEAT-01.SPEC-002-AC-04:** Given Sam opens a reset link that has already expired, when the screen loads, then he sees "This reset link is no longer valid. Request a new one." with a button to request a new link.

**FEAT-01.SPEC-002-AC-05:** Given Maya enters two different values in "New password" and "Confirm new password", when she blurs the confirm field, then the error "Passwords don't match" appears and the form does not submit.

**FEAT-01.SPEC-002-AC-06:** Given Maya has requested a reset and then requests a second one before using the first link, when she opens the first (now superseded) link, then it shows as no longer valid.

**FEAT-01.SPEC-002-AC-07:** Given Sam loses connectivity while on the request step, when he taps "Send reset link", then the banner "You're offline. Password reset needs a connection -- try again once you're back online." appears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (request, confirmation, reset, invalid link, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Household Naming & Guided Setup Start

## Overview

**Name:** Household Naming & Guided Setup Start
**ID:** FEAT-01.SPEC-003
**Type:** Screen
**Purpose:** The organiser names the household, becomes its first member, and enters the guided setup flow.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Naming the household and creating the Household record
- Creating the organiser's own Member Profile as the household's first member
- Introducing the guided setup wizard shell that carries through the following setup screens
- Triggering the default free-tier Subscription on household creation

**Non-Goals:**
- Adding other household members -- handled by FEAT-01.SPEC-004 (Member List & Add Member), the next guided-setup step
- Editing an existing household's name after setup -- handled by FEAT-01.SPEC-010 (Household Settings Hub); this screen only covers the one-time creation moment
- Choosing units, currency, or aisle layout -- owned by FEAT-16 (Units, Currency & Locale Configuration), reached later in the same guided setup journey (per the Internal Dependency Map's step 4)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | New account created, or sign-in for an account with no household yet | None -- form starts empty |
| FEAT-01.SPEC-002 (Password Recovery) | Reset completes for a household-less account | None |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser taps to edit the household name later | Existing household_name pre-filled; screen behaves in edit mode (see States) |
| FEAT-24 (Invite Another Household) | Referred visitor completes account creation and starts their own household | Referring household's Household Referral record is created once this step completes |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Name/rename the household, proceed into guided setup | -- |
| Sam (Other Adult Member) | No | No | Household naming is not exposed to other adult members; Sam has View-only access to household facts once they exist (FEAT-01.SPEC-010) |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View only, and only through FEAT-22 (from v1) | No | Riley never reaches this screen directly; household facts are visible only inside the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Entered household name is preserved locally (FEAT-01.SPEC-013) and restored after re-authentication |

## Layout and Content

**Header:** Guided setup wizard shell (shared with FEAT-01.SPEC-004 through FEAT-01.SPEC-009, continuing through FEAT-16.SPEC-001 and FEAT-16.SPEC-002 between the budget step and setup completion): step indicator "Step 1 of 8," screen title "Name your household."

**Body:** A single field, "Household name" (text input, required), with helper text: "This is what your household will be called inside Plateful -- you and everyone you invite will see it." Below the field, a "Continue" button.

In edit mode (arrived from FEAT-01.SPEC-010), the wizard chrome and step indicator are omitted; only the field, pre-filled with the current name, and a "Save" button appear, with a "Cancel" link returning to the Settings Hub.

**Footer:** None in guided-setup mode (Continue is the sole action); in edit mode, "Cancel" sits beside "Save."

### Responsive Behavior

- **Compact breakpoint:** Field and button stack full width.
- **Medium size class and above:** Content caps at the platform-wide narrow form width, horizontally centered; step indicator remains at the top, unchanged in structure.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Household name field | Type | Captures the household name | Field shows entered text; draft saved locally per FEAT-01.SPEC-013 | Standard input focus state |
| Household name field | Blur | Validates via FEAT-01.SPEC-014 | Error state if invalid | "Household name must be between 1 and 60 characters" below the field |
| "Continue" button (guided setup) | Tap | 1. Validate the name. 2. Create the Household record with the organiser as its first Member Profile. 3. Trigger FEAT-01.SPEC-011 (Default Subscription Provisioning). 4. The completed household creation triggers FEAT-24.SPEC-004 (Household Referral Recording), which records a referral only when referral context is present. | Button shows inline saving confirmation, not a full-page spinner | Success: navigates to FEAT-01.SPEC-004 (Member List & Add Member). Failure: inline error, entered name retained |
| "Save" button (edit mode) | Tap | Validate and update the Household's household_name field | Inline saving confirmation | Success: toast "Household name updated," returns to FEAT-01.SPEC-010. Failure: inline error, entered name retained |
| "Cancel" link (edit mode) | Tap | Discards the in-progress edit | Screen closes | Returns to FEAT-01.SPEC-010 without saving |

### Accessibility Notes

- **Focus order:** Household name field -> Continue/Save button (-> Cancel link, edit mode only).
- **Validation announcements:** The name-length error is announced and associated with the field when it appears.
- **Save feedback:** The "Household name updated" toast (edit mode) is announced on success.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (guided setup, default) | Field empty, Continue enabled | First-time arrival at this step | User begins typing |
| Filling | Field contains entered text | User types | User taps Continue/Save or navigates away |
| Saving | Inline saving confirmation shown, field disabled briefly | User taps Continue/Save with valid input | Save completes or fails |
| Error | Failed field highlighted with error message below it | Validation fails or save fails | User corrects the field and retries |
| Edit (pre-filled) | Field pre-filled with current household_name | Arrived from FEAT-01.SPEC-010 | User saves, cancels, or navigates away |
| Offline/Degraded | Banner: "You're offline -- this will save when you reconnect." Field remains editable; Continue/Save queues the change locally per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored -- queued save submits automatically and standard success feedback appears |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules). See that spec for the household name length rule. Checked on field blur and on submission.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful household creation (guided setup) | FEAT-01.SPEC-004 (Member List & Add Member) | -- |
| Successful save (edit mode) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |
| Cancel (edit mode) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |

## Data Model

**Creates:** Household -- household_name set from form input; organiser field set to the signed-in account's new Member Profile; weekly_budget, weekly_schedule, unit_system, currency, aisle_names, and plan_arrival_day_time left at their defaults or unset until later setup steps; status set to Active. Member Profile -- display_name (captured here as part of household creation, drawn from the organiser's account), member_type set to Organiser, status set to Active.
**Reads:** In edit mode, Household.household_name.
**Updates:** In edit mode, Household.household_name only.
**Deletes:** None.

## Business Rules

- Household creation is a one-time event per account (scope-boundaries.md SC-03) -- an account with a household already never sees this screen again except in edit mode.
- Household creation immediately triggers FEAT-01.SPEC-011 (Default Subscription Provisioning) so the household is never left without a Subscription record, even momentarily.
- If this creation completed via a household referral link (FEAT-24), the corresponding Household Referral record is attributed at this moment, per XBR-20.
- A failed save keeps the entered data on screen with a retry option -- nothing is silently discarded, per this feature's States field commitment.

## Edge Cases

- **User leaves the household name field empty and taps Continue** -- Error: "Household name must be between 1 and 60 characters"; household is not created.
- **User taps Continue twice rapidly** -- Second tap is ignored while the first creation request is in progress.
- **Network failure during household creation** -- Error banner with retry; the household name is preserved on screen and no partial Household record is left behind.
- **Organiser edits the household name from the Settings Hub while offline** -- The name change queues locally and saves automatically once connectivity returns, per FEAT-01.SPEC-013.
- **Concurrent edit from two devices (Maya signed in on laptop and phone)** -- Save is accepted on a last-write-wins basis per the dependency map's Contention note for Household: the most recently saved name wins, and a failed save on either device keeps the household's prior name active rather than leaving a mixed state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (inbound) | New or household-less accounts arrive here |
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (outbound) | Next step in guided setup |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Edit-mode entry and return |
| FEAT-01.SPEC-011 (Default Subscription Provisioning) | Triggers (outbound) | Household creation provisions the free-tier Subscription |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Draft saving and offline queuing behavior |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Household name validation |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated access to this screen |
| FEAT-24 (Invite Another Household) | Navigation (inbound) | Referral welcome page leads here via account creation |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| household_created | entry source (default / referral) | Household record is successfully created | supports success-metrics.md: "First-Session Onboarding Completion" |
| household_name_edited | -- | Organiser saves a name change from edit mode | N/A -- this event is a later edit, not part of the first-session onboarding window the connected metric measures |

## Acceptance Criteria

**FEAT-01.SPEC-003-AC-01:** Given Maya has just created her account, when she enters "The Rossi Household" and taps "Continue", then the Household is created with her as its first member and she is taken to FEAT-01.SPEC-004 (Member List & Add Member).

**FEAT-01.SPEC-003-AC-02:** Given Maya leaves the household name field empty, when she taps "Continue", then the error "Household name must be between 1 and 60 characters" appears and no household is created.

**FEAT-01.SPEC-003-AC-03:** Given Maya's household is created, then FEAT-01.SPEC-011 (Default Subscription Provisioning) fires automatically and provisions a free-tier Subscription.

**FEAT-01.SPEC-003-AC-04:** Given Maya opens this screen from FEAT-01.SPEC-010 to rename her household, when she changes the name and taps "Save", then the household's name updates and she sees the toast "Household name updated," returning to the Settings Hub.

**FEAT-01.SPEC-003-AC-05:** Given Maya is in edit mode with unsaved changes, when she taps "Cancel", then she returns to FEAT-01.SPEC-010 without the household name being changed.

**FEAT-01.SPEC-003-AC-06:** Given Maya loses connectivity while entering the household name, when she taps "Continue", then the banner "You're offline -- this will save when you reconnect." appears and the creation is queued locally.

**FEAT-01.SPEC-003-AC-07:** Given Maya is signed in on both a laptop and a phone, when she saves a different household name from each device within moments of each other, then the most recently saved name is the one that persists, per last-write-wins.

**FEAT-01.SPEC-003-AC-08:** Given a visitor arrived via a household referral link and creates their account, when they complete this screen, then a Household Referral record attributing their new household to the referring household is created per XBR-20.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (empty, filling, saving, error, edit, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Member List & Add Member

## Overview

**Name:** Member List & Add Member
**ID:** FEAT-01.SPEC-004
**Type:** Screen
**Purpose:** The organiser sees every household member and starts adding an adult or kid profile; other adult members view the list.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Listing every current Member Profile in the household
- Starting the add-adult or add-kid-profile flow (routing into FEAT-01.SPEC-005, with a detour through FEAT-01.SPEC-007 for kids)
- An entry point into inviting another adult (hands off to FEAT-09)
- Read-only viewing of the list for Sam (Other Adult Member)

**Non-Goals:**
- Editing an existing member's details -- handled by FEAT-01.SPEC-005 (Member Profile Detail), reached by tapping a member card
- Removing a member -- owned by Account & Data Management (FEAT-18) per the dependency map and XBR-16; this screen creates and lists members but never removes one
- Sending or managing the invitation itself -- owned by FEAT-09 (Household Invitations & Membership); this screen only offers the entry point

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Organiser continues from naming the household | Guided-setup wizard context (step indicator continues) |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser or Sam taps "Members" | None -- list opens in its standalone (non-wizard) chrome |
| FEAT-09 (Household Invitations & Membership) | An accepted invitation completes | The list refreshes to include the new Member Profile |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen, every member's card | Add an adult or kid profile, invite another adult, tap into any member's detail | -- |
| Sam (Other Adult Member) | Full screen, every member's card | View only -- Add and Invite controls are not shown; tapping a card opens the member's detail in read-only mode (kid profiles) or Sam's own detail (edit access to his own notification preferences only, per FEAT-01.SPEC-016) | Attempting to reach Add/Invite by direct navigation shows "Only the organiser can add members" and returns to this list |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1) | No | Riley never reaches this screen directly; the member list is visible only inside the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." No in-progress add-member entry exists on this list screen itself to preserve |

## Layout and Content

**Header (guided setup):** Wizard shell, "Step 2 of 8," title "Who's eating with you?" **Header (standalone, from Settings Hub):** Title "Members," back arrow to FEAT-01.SPEC-010.

**Body:** A vertical list of member cards, one per current Member Profile, each showing: display name, member type (Organiser / Other Adult Member / Kid), and up to three dietary-rule badges (e.g., "Peanut allergy," "Vegetarian") summarizing that member's rules. Below the list, two actions for the organiser only: "Add an adult," "Add a kid profile." A third action, "Invite a partner" (visible to the organiser only), hands off to FEAT-09.

For Sam, the same list renders without the Add/Invite actions beneath it.

An empty state (only the organiser's own card exists) shows a short prompt above the actions: "Add the people eating with you."

**Footer (guided setup only):** "Continue" button, enabled once at least the organiser's own profile exists (always true by this point).

### Responsive Behavior

- **Compact breakpoint:** Member cards stack full width, one per row; action buttons stack full width below the list.
- **Medium size class and above:** Member cards render in a two-column grid; action buttons remain full width in a row beneath the grid.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Member card | Tap | Navigate to FEAT-01.SPEC-005 (Member Profile Detail) for that member | Screen changes | Standard transition; Sam's tap opens the same screen in his role's view/edit scope |
| "Add an adult" (organiser only) | Tap | Navigate to FEAT-01.SPEC-005 in create mode, member_type pre-set to Other Adult Member | Screen changes | Standard transition |
| "Add a kid profile" (organiser only) | Tap | Navigate to FEAT-01.SPEC-007 (Parental Consent Confirmation) first | Screen changes | Standard transition; kid creation always detours through consent first |
| "Invite a partner" (organiser only) | Tap | Navigate to FEAT-09 (Household Invitations & Membership) | Screen changes | Standard transition, leaving this feature |
| "Continue" button (guided setup) | Tap | Proceeds to the next guided-setup step | Screen changes | Standard transition to FEAT-01.SPEC-008 |

### Accessibility Notes

- **Focus order:** Member cards in list order -> "Add an adult" -> "Add a kid profile" -> "Invite a partner" (organiser only) -> Continue (guided setup only).
- **Dynamic update announcement:** When a member is added and the list refreshes (returning from FEAT-01.SPEC-005 or an accepted invitation), the new card's addition is announced ("{display name} added").
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Minimal (organiser only) | One card (the organiser), prompt "Add the people eating with you." | First arrival in guided setup | A member is added |
| Populated | Two or more member cards | At least one member beyond the organiser exists | Always the state once populated |
| Loading | Cards render with a brief inline placeholder | List first loads (standalone entry from Settings Hub) | Data loads (typically under a second) |
| Error | Banner: "Couldn't load your household's members. Try again." with retry | Member data fails to load | Retry succeeds |
| Offline/Degraded | Previously loaded list remains fully viewable; Add actions remain reachable, entering FEAT-01.SPEC-005/007 which queue their own saves per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules), specifically the 12-member household cap enforced when "Add an adult" or "Add a kid profile" is attempted at the limit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Member card tap | FEAT-01.SPEC-005 (Member Profile Detail) | -- |
| "Add an adult" tap | FEAT-01.SPEC-005 (Member Profile Detail, create mode) | -- |
| "Add a kid profile" tap | FEAT-01.SPEC-007 (Parental Consent Confirmation) | -- |
| "Invite a partner" tap | FEAT-09 (Household Invitations & Membership) | FEAT-09 |
| "Continue" tap (guided setup) | FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | -- |
| Back arrow (standalone) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |

## Data Model

**Creates:** None directly -- member creation happens on FEAT-01.SPEC-005/007.
**Reads:** Member Profile -- display_name, member_type, status (Active members only), plus a summary of each member's Dietary Rule badges, for every member of the household.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Attempting to add a 13th member profile is blocked per FEAT-01.SPEC-014 with a message citing the 12-member cap; the Add actions remain visible but the resulting create screen shows the block.
- Only the organiser can add members or send invitations (FEAT-01.SPEC-016).
- A member added via accepted invitation (FEAT-09) appears in this list automatically without the organiser taking any action here.

## Edge Cases

- **Household is at the 12-member cap** -- "Add an adult" and "Add a kid profile" remain visible (they are not hidden); tapping either surfaces the cap message from FEAT-01.SPEC-014 rather than opening the create screen.
- **Sam attempts to reach the add-member screen directly (e.g., a stale link)** -- He is shown "Only the organiser can add members" and returned to this list, per FEAT-01.SPEC-016.
- **An invitation is accepted while Maya has this screen open** -- The list updates to include the new member without requiring a manual refresh (live-updating, not a snapshot), consistent with the dependency map's low-contention note for Member Profile creation.
- **Network failure while loading the list** -- Error banner with retry; no member cards are shown until the retry succeeds.
- **Organiser navigates away and back mid-guided-setup** -- The list re-fetches current members; nothing added so far is lost.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Navigation (inbound) | Guided setup continues here |
| FEAT-01.SPEC-005 (Member Profile Detail) | Navigation (outbound) | Add-adult and member-detail entry |
| FEAT-01.SPEC-007 (Parental Consent Confirmation) | Navigation (outbound) | Add-kid-profile entry point |
| FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | Navigation (outbound) | Next guided-setup step |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Standalone entry and return |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Offline behavior for downstream add flows |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | 12-member cap enforcement |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated Add/Invite visibility |
| FEAT-09 (Household Invitations & Membership) | Navigation (outbound); Navigation (inbound) | Invite entry point; accepted invitations refresh this list |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| member_list_viewed | member count, role of viewer | Screen opens | N/A -- no Stage 2 metric measures list views directly; retained as a funnel step for First-Session Onboarding Completion analysis |
| member_added | member type (adult / kid) | A member is successfully created (event actually recorded by FEAT-01.SPEC-005/FEAT-01.SPEC-007, surfaced here as the list's refresh trigger) | supports success-metrics.md: "First-Session Onboarding Completion" |

## Acceptance Criteria

**FEAT-01.SPEC-004-AC-01:** Given Maya has just named her household, when this screen loads, then she sees her own member card and the "Add an adult," "Add a kid profile," and "Invite a partner" actions.

**FEAT-01.SPEC-004-AC-02:** Given Maya taps "Add an adult", then she is taken to FEAT-01.SPEC-005 in create mode with member_type pre-set to Other Adult Member.

**FEAT-01.SPEC-004-AC-03:** Given Maya taps "Add a kid profile", then she is taken to FEAT-01.SPEC-007 (Parental Consent Confirmation) before any kid profile detail screen.

**FEAT-01.SPEC-004-AC-04:** Given Sam opens this screen, when it loads, then he sees every member card but no "Add" or "Invite" actions.

**FEAT-01.SPEC-004-AC-05:** Given Maya's household already has 12 members, when she taps "Add an adult", then the resulting screen shows the 12-member-cap message from FEAT-01.SPEC-014 and no 13th member is created.

**FEAT-01.SPEC-004-AC-06:** Given Sam attempts to navigate directly to the add-member screen, then he is shown "Only the organiser can add members" and returned to this list.

**FEAT-01.SPEC-004-AC-07:** Given Maya has this screen open, when Sam's invited partner accepts their invitation via FEAT-09, then the new member's card appears on Maya's list without her needing to refresh manually.

**FEAT-01.SPEC-004-AC-08:** Given the member list fails to load, when the screen opens, then the banner "Couldn't load your household's members. Try again." appears with a retry option.

**FEAT-01.SPEC-004-AC-09:** Given Maya is offline, when she taps "Add an adult", then she is still taken to FEAT-01.SPEC-005, which queues the save per FEAT-01.SPEC-013.

**FEAT-01.SPEC-004-AC-10:** Given Maya has added all household members in guided setup, when she taps "Continue", then she is taken to FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (minimal, populated, loading, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Member Profile Detail

## Overview

**Name:** Member Profile Detail
**ID:** FEAT-01.SPEC-005
**Type:** Screen
**Purpose:** The organiser creates or edits one member's basic details -- name, type, age band, and notification preferences; other adult members view a kid profile's details.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Creating a new adult Member Profile (kid profiles arrive here only after FEAT-01.SPEC-007 confirms consent)
- Editing an existing member's display name, member type details, age band, and notification preferences
- Read-only viewing of a kid profile's details for Sam
- Entry point into FEAT-01.SPEC-006 (Dietary Rules Editor) for the member being viewed or edited

**Non-Goals:**
- Setting or editing dietary rules -- handled by FEAT-01.SPEC-006 (Dietary Rules Editor), reached from this screen
- Parental consent confirmation itself -- handled by FEAT-01.SPEC-007, which this screen is routed through when the member type is a kid profile
- Removing a member -- owned by Account & Data Management (FEAT-18); this screen creates and edits, never deletes

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser taps "Add an adult" | member_type pre-set to Other Adult Member; screen opens in create mode, empty |
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser or Sam taps an existing member's card | The member's current data, loaded editable for the organiser or read-only for Sam viewing a kid profile |
| FEAT-01.SPEC-007 (Parental Consent Confirmation) | Organiser confirms consent for a new kid profile | member_type pre-set to Kid; screen opens in create mode with the consent already confirmed |
| FEAT-01.SPEC-010 (Household Settings Hub) | Any adult taps "Notification preferences" | The signed-in adult's own Member Profile, scrolled to the Notification preferences toggles |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen for every member, including create mode | Create, edit any member's display name, type details, age band, and notification preferences | -- |
| Sam (Other Adult Member) | Full screen, read-only, for kid profiles; his own detail shows his own notification preferences as editable | View kid profiles; edit only his own notification preferences (per FEAT-01.SPEC-016) | Attempting to edit a kid profile's fields shows the fields as non-interactive with the note "Only the organiser can change member details" |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1), and never kid profile data beyond a specific safety report (XBR-14) | No | Riley never reaches this screen directly |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Unsaved edits are preserved locally per FEAT-01.SPEC-013 and restored after re-authentication |

## Layout and Content

**Header:** Title "{display name}" (or "New adult" / "New kid profile" in create mode), back arrow returning to FEAT-01.SPEC-004.

**Body:** A single-column form:
- Display name (text input, required)
- Member type (read-only label once set: "Organiser," "Other Adult Member," or "Kid" -- not editable after creation, since type determines the fields below)
- Age band (selection input, kid profiles only, required for kids)
- Notification preferences: "Plan-ready notifications" (toggle) and "Nightly nudge" (toggle) -- adults only, and each adult can edit only their own
- A "Dietary rules" summary row showing up to three rule badges and a "View/Edit dietary rules" link into FEAT-01.SPEC-006

For a kid profile, a note appears beneath the header: "Stored for {display name}: first name or nickname, age band, and dietary rules only," restating the data-minimality commitment from FEAT-01.SPEC-007.

**Footer:** "Save" button (create and edit modes); "Cancel" link discarding unsaved changes.

### Responsive Behavior

- **Compact breakpoint:** Fields stack full width; footer buttons stack full width.
- **Medium size class and above:** Form caps at the platform-wide narrow form width, horizontally centered; footer buttons appear side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Display name field | Type | Captures the display name; draft saved locally per FEAT-01.SPEC-013 | Field shows entered text | Standard input focus state |
| Display name field | Blur | Validates via FEAT-01.SPEC-014 | Error state if invalid | "Enter a name (up to 60 characters)" below the field |
| Age band selector (kid profiles) | Select | Sets the age band | Selector shows chosen band | Selected band displayed |
| Notification toggles (own profile only) | Tap | Sets the toggle on/off | Toggle state changes visually | Immediate; no separate save step required for preference toggles once the profile already exists |
| "View/Edit dietary rules" link | Tap | Navigate to FEAT-01.SPEC-006 (Dietary Rules Editor) for this member | Screen changes | Standard transition |
| "Save" button | Tap | 1. Validate all fields via FEAT-01.SPEC-014. 2. Create or update the Member Profile. | Button shows inline saving confirmation | Success: returns to FEAT-01.SPEC-004 with the updated/new card visible. Failure: inline error, entered data retained |
| "Cancel" link | Tap | Discards unsaved changes | Screen closes | Confirmation dialog if changes were made; returns to FEAT-01.SPEC-004 |

### Accessibility Notes

- **Focus order:** Display name -> Member type (read-only, not focusable) -> Age band (kid profiles) -> Notification toggles (own profile) -> Dietary rules link -> Save -> Cancel.
- **Validation announcements:** Field errors are announced and associated with their field.
- **Read-only announcement (Sam viewing a kid profile):** Fields are announced as "read-only" to assistive technology so their non-interactive state is not silently assumed to be a bug.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Create (empty) | Form fields empty except member_type (pre-set), Save enabled once required fields are filled | Arrived from "Add an adult" or FEAT-01.SPEC-007 | User fills required fields and saves, or cancels |
| Edit (pre-filled) | Form fields show the member's current data | Arrived by tapping an existing member's card (organiser) | User saves or cancels |
| Read-only (Sam viewing a kid profile) | Fields shown as static text/labels, no inputs | Sam taps a kid profile's card | Sam navigates away |
| Saving | Inline saving confirmation, fields disabled briefly | User taps Save with valid input | Save completes or fails |
| Error | Failed field highlighted with error message; entered data retained | Validation fails or save fails | User corrects and retries |
| Offline/Degraded | Banner: "You're offline -- this will save when you reconnect." Fields remain editable; Save queues the change locally per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored -- queued save submits automatically |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules). See that spec for display name length, age band requirement for kid profiles, and kid-profile data-minimality rules. Checked on field blur and on submission.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful save | FEAT-01.SPEC-004 (Member List & Add Member) | -- |
| Cancel | FEAT-01.SPEC-004 (Member List & Add Member) | -- |
| "View/Edit dietary rules" tap | FEAT-01.SPEC-006 (Dietary Rules Editor) | -- |

## Data Model

**Creates:** Member Profile -- display_name, member_type, age_band (kid profiles), status set to Active. For kid profiles, parental_consent_confirmation is already set to true by FEAT-01.SPEC-007 before this screen is reached in create mode.
**Reads:** Member Profile -- all fields, for the member being viewed/edited; Dietary Rule -- summary badges for the dietary rules row.
**Updates:** Member Profile -- display_name, age_band (kid profiles), notification_preferences (adults, own-only).
**Deletes:** None.

## Business Rules

- Member type cannot be changed after creation -- a profile created as a kid cannot become an adult or vice versa, since the two carry fundamentally different field sets and consent requirements; correcting a mis-typed profile requires removing it (FEAT-18) and re-adding it correctly.
- A kid profile's fields are restricted to display_name, member_type, age_band, dietary rules, and parental_consent_confirmation -- no surname, birth date, photo, or contact detail field exists on this screen for kid profiles, per FEAT-01.SPEC-014's data-minimality rule.
- Each adult edits only their own notification preferences (FEAT-01.SPEC-016) -- the organiser cannot set another adult's toggle on this screen.
- Authorization for who can create, view, or edit is governed by FEAT-01.SPEC-016 (Household Setup Authorization Rules).

## Edge Cases

- **Organiser leaves the display name empty and taps Save** -- Error: "Enter a name (up to 60 characters)"; profile is not created/saved.
- **Organiser attempts to add a 13th member** -- Blocked per FEAT-01.SPEC-014's 12-member cap, shown as an inline message on this screen before Save is enabled.
- **Organiser navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Sam attempts to edit a kid profile's display name via direct interaction** -- The field is non-interactive; no error is needed since there is no control to trigger one.
- **Concurrent edit from two devices (Maya on laptop and phone, editing the same member)** -- Save is rejected with reject-with-refresh, consistent with the dependency map's Contention note for Member Profile: dialog "This profile was updated from your other device. Review the latest version before saving." with "View Latest" and "Keep Editing" options.
- **Network failure during save** -- Error banner with retry; entered data preserved, no partial Member Profile is left behind.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (inbound/outbound) | Entry and return |
| FEAT-01.SPEC-006 (Dietary Rules Editor) | Navigation (outbound) | Dietary rules link |
| FEAT-01.SPEC-007 (Parental Consent Confirmation) | Navigation (inbound) | Kid profile creation arrives here after consent |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Draft saving and offline behavior |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Field validation and the 12-member cap |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated view/edit access |
| FEAT-07 (Weekly Plan Ready Notification) | References (outbound) | Plan-ready preference toggle exposed here |
| FEAT-13 (Tonight's Dinner Reminder) | References (outbound) | Nightly nudge preference toggle exposed here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| member_added | member type (adult / kid) | New Member Profile is successfully created | supports success-metrics.md: "First-Session Onboarding Completion" |
| member_profile_edited | fields changed (name / age band / notification preference) | An existing member's details are saved | N/A -- this event covers post-setup edits, outside the first-session onboarding window the connected metric measures |

## Acceptance Criteria

**FEAT-01.SPEC-005-AC-01:** Given Maya taps "Add an adult" from FEAT-01.SPEC-004, when she enters "Sam" and taps "Save", then a new Other Adult Member profile named "Sam" is created and she returns to the member list showing his card.

**FEAT-01.SPEC-005-AC-02:** Given Maya arrives here after confirming parental consent (FEAT-01.SPEC-007) for a kid, when she enters "Jordan" and selects an age band and taps "Save", then a new kid profile is created with only the name, member type, age band, and consent recorded.

**FEAT-01.SPEC-005-AC-03:** Given Maya leaves the display name empty, when she taps "Save", then the error "Enter a name (up to 60 characters)" appears and no profile is created.

**FEAT-01.SPEC-005-AC-04:** Given Sam taps Jordan's member card from the list, when the screen loads, then all fields render as read-only static text and no edit controls are shown.

**FEAT-01.SPEC-005-AC-05:** Given Sam opens his own member detail, when he toggles "Nightly nudge" off, then his own preference updates and Maya's preferences are unaffected.

**FEAT-01.SPEC-005-AC-06:** Given Maya is editing an existing member with unsaved changes, when she taps "Cancel", then a confirmation dialog "You have unsaved changes. Discard?" appears.

**FEAT-01.SPEC-005-AC-07:** Given Maya is signed in on two devices and edits the same member's age band from both within moments, when the second save is submitted, then it is rejected with "This profile was updated from your other device. Review the latest version before saving."

**FEAT-01.SPEC-005-AC-08:** Given Maya's household already has 12 members, when she attempts to save a 13th, then the 12-member-cap message from FEAT-01.SPEC-014 is shown and no profile is created.

**FEAT-01.SPEC-005-AC-09:** Given Maya loses connectivity while editing a member, when she taps "Save", then the banner "You're offline -- this will save when you reconnect." appears and the change is queued locally.

**FEAT-01.SPEC-005-AC-10:** Given Maya is viewing any member's detail, when she taps "View/Edit dietary rules", then she is taken to FEAT-01.SPEC-006 for that specific member.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (create, edit, read-only, saving, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Dietary Rules Editor

## Overview

**Name:** Dietary Rules Editor
**ID:** FEAT-01.SPEC-006
**Type:** Screen
**Purpose:** The organiser records or edits one member's allergies, religious rules, vegetarian setting, and dislikes, and sees each rule's change history; other adult members view a kid profile's dietary rules.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Adding, editing, and viewing a member's Dietary Rule entries: allergies (from a standard list plus an optional named extra ingredient), religious rules, a per-person vegetarian setting, and dislikes
- Showing each rule's change history (who changed it and when)
- Removing a dislike directly, and routing an allergy or religious-rule removal through the explicit confirmation gate
- Read-only viewing of a kid profile's dietary rules for Sam

**Non-Goals:**
- Allergen classification logic, hard-vs-soft strength rules, and the removal confirmation gate itself -- governed by FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules); this screen is the surface, not the rulebook
- Checking a rule against any recipe -- owned by the Dietary Rules & Allergy Safety Engine (FEAT-02), which reads this data
- Medical or diet advice of any kind -- excluded per scope-boundaries.md SC-06: this screen records rules for matching purposes only, never as nutritional guidance

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-005 (Member Profile Detail) | Organiser or Sam taps "View/Edit dietary rules" | The member whose rules are being viewed/edited |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen for every member | Add, edit, and remove any member's dietary rules, subject to the removal confirmation gate | -- |
| Sam (Other Adult Member) | Full screen, read-only, for any member's rules | View only -- no add/edit/remove controls | Attempting to interact with a rule shows it as non-interactive with the note "Only the organiser can change dietary rules" |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- dietary rule administration is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1), and only the allergy details inside a specific open safety report -- never the full dietary-rule list at large (XBR-14) | No | Riley never reaches this screen directly; a safety report's allergy detail is surfaced only within the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Unsaved rule entry is preserved locally per FEAT-01.SPEC-013 |

## Layout and Content

**Header:** Title "{display name}'s dietary rules," back arrow returning to FEAT-01.SPEC-005.

**Body:** Grouped list of the member's current rules, organized by kind: Allergies, Religious Rules, Vegetarian Setting, Dislikes. Each rule row shows its label (e.g., "Peanuts (allergy)"), a "History" link opening the change_history for that rule inline, and (organiser only) edit and remove controls. A single "Add a rule" button opens a rule-entry form below the list (or as an expanding panel):
- Rule kind selector (Allergy / Religious rule / Vegetarian setting / Dislike)
- For Allergy: a selector from the standard allergen list, plus an optional "Name a specific ingredient" text field
- For Religious rule and Dislike: a free-text label field
- For Vegetarian setting: a single toggle (this member is vegetarian) plus, if the household shares meals, a note that a vegetarian variant will be offered where relevant

For a kid profile, the same layout renders without edit controls for Sam, with the data-minimality note carried from FEAT-01.SPEC-005/007 restated at the top: "{display name}'s stored data: dietary rules only, alongside their name and age band."

**Footer:** None -- each rule saves independently as it is added or edited, rather than through a single form-wide Save.

### Responsive Behavior

- **Compact breakpoint:** Rule groups stack full width; the add-rule panel expands full width below the list.
- **Medium size class and above:** Rule groups render in a single column capped at the platform-wide form width, horizontally centered; the add-rule panel appears as an inline expansion rather than a separate overlay.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Add a rule" button (organiser only) | Tap | Opens the rule-entry panel | Panel expands | Standard expand animation |
| Rule kind selector | Select | Shows the fields relevant to that kind | Form fields change | Immediate |
| Allergen selector | Select | Sets the allergen from the standard list | Selector shows chosen allergen | Selected allergen displayed |
| "Name a specific ingredient" field | Type | Captures the named ingredient | Field shows entered text | Standard input focus state |
| "Save" (within the add-rule panel) | Tap | 1. Validate via FEAT-01.SPEC-015. 2. Create the Dietary Rule with origin set to organiser-entered. | Panel closes, list updates | New rule appears in its group with a change_history entry ("Added by Maya, {date}") |
| Rule row "History" link | Tap | Expands the change_history for that rule inline | Row expands | Shows a list of "{who} changed {what}, {when}" entries |
| Rule row "Edit" (organiser only) | Tap | Opens the rule for editing in the add-rule panel, pre-filled | Panel expands with current values | Standard expand animation |
| Rule row "Remove" (dislike, organiser only) | Tap | Removes the dislike immediately (hard delete) | Row disappears from the list | Toast: "{rule} removed" |
| Rule row "Remove" (allergy or religious rule, organiser only) | Tap | Opens the removal confirmation modal per FEAT-01.SPEC-015 | Modal appears | Modal states what will happen and requires an explicit affirmative tap before removal proceeds |

### Accessibility Notes

- **Focus order:** Rule groups in order (Allergies -> Religious Rules -> Vegetarian -> Dislikes) -> "Add a rule" -> (within the panel) Rule kind -> kind-specific fields -> Save.
- **Change announcements:** Adding, editing, or removing a rule is announced ("{rule} added" / "{rule} removed") so the list's live change is not silently missed.
- **History disclosure:** Expanding a rule's history is announced as an expansion, and its content is read in order.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (no rules yet) | Grouped headers with "No rules recorded yet" under each, "Add a rule" prominent | Member has no Dietary Rule entries | A rule is added |
| Populated | Grouped rule list with existing entries | One or more rules exist | Always the state once populated |
| Adding/Editing | Add-rule panel expanded with fields for the selected kind | User taps "Add a rule" or a rule's "Edit" | User saves or cancels the panel |
| Removal confirmation pending | Confirmation modal shown per FEAT-01.SPEC-015 | User taps "Remove" on an allergy or religious rule | User confirms (rule removed) or cancels (modal closes, rule unchanged) |
| Error | Inline error on the add-rule panel or a toast for a failed remove; entered data retained | A save or remove operation fails | User retries |
| Offline/Degraded | Banner: "You're offline -- rule changes will save when you reconnect." Add/edit remain usable and queue locally per FEAT-01.SPEC-013; removal of an allergy or religious rule is deferred until connectivity returns, since it requires the confirmation gate to complete against the current record | Connectivity lost while this screen is open | Connectivity restored -- queued changes save automatically |

## Validation Rules

Validation governed by FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules). See that spec for allergen selection rules, hard/soft strength assignment, vegetarian logic, and the removal confirmation gate. Checked on save of each rule entry.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-005 (Member Profile Detail) | -- |

## Data Model

**Creates:** Dietary Rule -- member (set to this screen's member), rule_kind, strength (derived per FEAT-01.SPEC-015), allergen (allergies only), origin set to "entered by the organiser," change_history initialized with the creation entry.
**Reads:** Dietary Rule -- all fields and change_history, for every rule belonging to this member.
**Updates:** Dietary Rule -- rule_kind-specific fields (allergen, named ingredient, strength where editable) on edit; change_history appended on every change.
**Deletes:** Dietary Rule -- dislikes removed directly; allergies and religious rules removed only after the FEAT-01.SPEC-015 confirmation gate. The change_history entry for a removed rule is retained per this feature's Data Notes (audit trail), even though the rule itself is gone.

## Business Rules

- A new or tightened hard rule (allergy or religious rule) triggers FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger), which re-checks the current week's plan immediately.
- Every dietary rule recorded here is read by the Dietary Rules & Allergy Safety Engine (FEAT-02) against every candidate recipe's ingredients.
- Rule classification, strength, and the removal confirmation gate are governed entirely by FEAT-01.SPEC-015 -- this screen never duplicates that logic.
- Change history is visible to the organiser for every rule and is never deleted even when the rule itself is removed, supporting trust and any safety investigation.

## Edge Cases

- **Organiser attempts to save an allergy with no allergen selected** -- Blocked per FEAT-01.SPEC-015 with the error defined there; no rule is created.
- **Organiser removes a dislike, then immediately adds it back** -- Treated as two independent operations: a new Dietary Rule is created with a fresh change_history, not a restoration of the removed one (no restore path, per this feature's Entity-Lifecycle Coverage Matrix).
- **Organiser attempts to remove an allergy while offline** -- The removal confirmation cannot complete against the current record while offline; the "Remove" action shows "Removing an allergy needs a connection -- try again once you're back online" instead of opening the confirmation modal.
- **Two rules for the same allergen entered in quick succession** -- The second entry is treated as an edit target, not a duplicate: the add-rule panel surfaces the existing allergen entry for editing rather than creating a second Dietary Rule for the same allergen.
- **Concurrent edit from two devices (Maya on laptop and phone)** -- Rule creation and edits are last-write-wins per field, consistent with the dependency map's Contention note for Dietary Rule; an allergy can never be silently dropped by this path since removal always requires the explicit confirmation gate.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-005 (Member Profile Detail) | Navigation (inbound/outbound) | Entry and return |
| FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) | Triggers (outbound) | A new/tightened hard rule fires the mid-week re-check |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Draft saving and offline queuing |
| FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules) | References (inbound) | All classification, strength, and removal-gate logic |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated view/edit access |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Affects (outbound) | Every rule recorded here is read by the safety engine |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| dietary_rule_added | rule kind (allergy / religious / vegetarian / dislike) | A new rule is saved | supports success-metrics.md: "First-Session Onboarding Completion" (dietary rules are part of the setup this metric measures completing within the first session) |
| dietary_rule_changed | rule kind, change type (edit / remove) | An existing rule is edited or removed | N/A -- occurs after first-session setup in most cases; retained as an operational signal, not tied to a Stage 2 metric for this feature |

## Acceptance Criteria

**FEAT-01.SPEC-006-AC-01:** Given Maya is on Jordan's Dietary Rules Editor with no rules yet, when she taps "Add a rule", selects "Allergy," chooses "Peanuts" from the standard list, and saves, then a new hard allergy rule appears under Allergies with a change_history entry "Added by Maya, {today's date}".

**FEAT-01.SPEC-006-AC-02:** Given Maya adds an allergy with no allergen selected, when she taps Save, then the rule is not created and the error defined by FEAT-01.SPEC-015 is shown.

**FEAT-01.SPEC-006-AC-03:** Given Maya taps "Remove" on a dislike, then the dislike is removed immediately with the toast "{rule} removed" and no confirmation step.

**FEAT-01.SPEC-006-AC-04:** Given Maya taps "Remove" on an existing peanut allergy, then the removal confirmation modal from FEAT-01.SPEC-015 appears and the allergy is not removed until she taps the explicit affirmative action.

**FEAT-01.SPEC-006-AC-05:** Given Sam opens Jordan's dietary rules, when the screen loads, then every rule renders read-only with no edit or remove controls.

**FEAT-01.SPEC-006-AC-06:** Given Maya taps "History" on an existing rule, then the rule's change_history expands inline showing who changed it and when.

**FEAT-01.SPEC-006-AC-07:** Given Maya adds a new hard allergy rule for Jordan, then FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) fires to re-check the current week's plan.

**FEAT-01.SPEC-006-AC-08:** Given Maya is offline and taps "Remove" on an allergy, then she sees "Removing an allergy needs a connection -- try again once you're back online" and no confirmation modal opens.

**FEAT-01.SPEC-006-AC-09:** Given Maya adds a dislike while offline, then the entry is queued locally per FEAT-01.SPEC-013 and saves automatically once connectivity returns.

**FEAT-01.SPEC-006-AC-10:** Given Maya sets the vegetarian toggle for a member on, when she saves, then the vegetarian rule is recorded for that member and the household is told a vegetarian variant will be offered on shared meals.

**FEAT-01.SPEC-006-AC-11:** Given Maya is signed in on two devices and edits the same member's dislike list from both within moments, then the changes apply last-write-wins per field, and no allergy is ever silently dropped by this path.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 6 (empty, populated, adding/editing, removal confirmation, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Parental Consent Confirmation

## Overview

**Name:** Parental Consent Confirmation
**ID:** FEAT-01.SPEC-007
**Type:** Screen
**Purpose:** The organiser confirms parent-or-guardian status and sees exactly what a kid profile stores before it is created.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Disclosing exactly what data a kid profile stores before any of it is entered
- Requiring an explicit affirmative confirmation that the organiser is the child's parent or guardian
- Blocking kid-profile creation until this confirmation is given

**Non-Goals:**
- Collecting the kid profile's actual data (name, age band, dietary rules) -- handled by FEAT-01.SPEC-005 and FEAT-01.SPEC-006, reached only after this confirmation
- Any legal or identity verification of parental status beyond the organiser's own affirmative confirmation -- product-features.md defines no identity-verification capability; the product relies on the organiser's stated confirmation, consistent with a household-trust model rather than a verification service
- Consent for an existing kid profile's continued data use -- this screen is a one-time gate at creation; ongoing data handling is governed by this feature's Data Notes and assumptions-constraints.md, not re-confirmed here

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser taps "Add a kid profile" | None -- confirmation is requested before any kid data is entered |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Confirm parent/guardian status and proceed, or cancel | -- |
| Sam (Other Adult Member) | No | No | This screen is reached only through the organiser's own "Add a kid profile" action; Sam has no entry point to it |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | No | No | N/A -- this is a one-time creation gate with no ongoing data to view; it is never exposed through operator support access |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." The intent to add a kid profile is not preserved as draft data (nothing has been entered yet); the organiser restarts from FEAT-01.SPEC-004 after re-authenticating |

## Layout and Content

**Header:** Title "Before you add a kid profile."

**Body:** A disclosure block listing exactly what will be stored, as a short bulleted list: "A first name or nickname," "An age band," "Their dietary rules (allergies, religious rules, vegetarian setting, dislikes)." Directly beneath, a plain statement: "We never store a surname, birth date, photo, or contact detail for a kid profile." Below that, a single checkbox with the label: "I confirm I am this child's parent or legal guardian." A "Continue" button, disabled until the checkbox is checked.

**Footer:** "Cancel" link, returning to FEAT-01.SPEC-004 without creating anything.

### Responsive Behavior

- **Compact breakpoint:** Disclosure list and checkbox stack full width; Continue and Cancel stack full width.
- **Medium size class and above:** Content caps at the platform-wide narrow form width, horizontally centered; Continue and Cancel appear side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Confirmation checkbox | Tap | Toggles the confirmation | Checkbox shows checked/unchecked; Continue enables/disables accordingly | Immediate visual toggle |
| "Continue" button | Tap (enabled only when checkbox is checked) | Records parental_consent_confirmation as true for the profile about to be created and proceeds | Screen changes | Navigates to FEAT-01.SPEC-005 in create mode, member_type pre-set to Kid |
| "Cancel" link | Tap | Discards the in-progress kid-add intent | Screen closes | Returns to FEAT-01.SPEC-004; nothing is created |

### Accessibility Notes

- **Focus order:** Disclosure list (read order) -> Confirmation checkbox -> Continue -> Cancel.
- **Disabled-state announcement:** The "Continue" button's disabled state is announced along with the reason ("Continue is disabled until you confirm parent or guardian status") so the requirement is not silently invisible to assistive technology.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Unconfirmed (default) | Checkbox unchecked, Continue disabled | Screen first opens | Checkbox is checked |
| Confirmed | Checkbox checked, Continue enabled | User checks the box | User taps Continue or unchecks the box |
| Offline/Degraded | N/A -- confirming and proceeding requires no network request; this screen only sets local state carried into FEAT-01.SPEC-005, whose own save is what queues offline | Connectivity lost while this screen is open | Connectivity has no effect on this screen's own function |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Confirmation checkbox | Must be checked before proceeding | On "Continue" tap (button is disabled otherwise, so no error state is reachable) | N/A -- the disabled button state itself prevents an invalid submission |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Continue" tap (confirmed) | FEAT-01.SPEC-005 (Member Profile Detail, create mode, Kid) | -- |
| "Cancel" tap | FEAT-01.SPEC-004 (Member List & Add Member) | -- |

## Data Model

**Creates:** None directly -- this screen sets the parental_consent_confirmation value carried into the Member Profile that FEAT-01.SPEC-005 creates immediately afterward.
**Reads:** None.
**Updates:** None.
**Deletes:** None.

## Business Rules

- A kid profile cannot exist without this confirmation -- FEAT-01.SPEC-005 cannot reach its Kid create mode by any path other than through this screen.
- The disclosure text on this screen is the authoritative statement of what a kid profile stores; FEAT-01.SPEC-005 and FEAT-01.SPEC-006 must not collect any field beyond what is disclosed here (enforced by FEAT-01.SPEC-014's kid-profile data-minimality rule).
- This screen is shown every time a kid profile is added, not only the first time -- each kid profile gets its own explicit confirmation.

## Edge Cases

- **Organiser checks the box, then unchecks it before tapping Continue** -- "Continue" disables again immediately; no state persists from the momentary check.
- **Organiser navigates away after checking the box but before tapping Continue** -- No confirmation dialog is needed; nothing has been created or saved, so there is no unsaved-data loss to warn about.
- **Organiser adds a second kid profile in the same session** -- This screen is shown again in full, from Unconfirmed, for the new profile; a prior confirmation for a different child does not carry over.
- **Double-tap on "Continue"** -- Second tap is ignored once navigation to FEAT-01.SPEC-005 has begun.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (inbound/outbound) | Entry via "Add a kid profile"; Cancel returns here |
| FEAT-01.SPEC-005 (Member Profile Detail) | Navigation (outbound) | Confirmed consent proceeds to kid profile creation |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (outbound) | The kid-profile data-minimality rule this screen's disclosure reflects |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| kid_profile_consent_confirmed | -- | Organiser taps "Continue" with the checkbox checked | supports success-metrics.md: "First-Session Onboarding Completion" |
| kid_profile_consent_cancelled | -- | Organiser taps "Cancel" | N/A -- no Stage 2 metric measures abandoned kid-profile starts; retained as a diagnostic signal for setup friction |

## Acceptance Criteria

**FEAT-01.SPEC-007-AC-01:** Given Maya taps "Add a kid profile" from FEAT-01.SPEC-004, when this screen loads, then she sees the exact disclosure list of what will be stored before any kid data field is shown.

**FEAT-01.SPEC-007-AC-02:** Given Maya has not checked the confirmation checkbox, when she looks at "Continue", then it is disabled.

**FEAT-01.SPEC-007-AC-03:** Given Maya checks "I confirm I am this child's parent or legal guardian" and taps "Continue", then she is taken to FEAT-01.SPEC-005 in create mode with member_type set to Kid, and the profile's parental_consent_confirmation will be recorded as true.

**FEAT-01.SPEC-007-AC-04:** Given Maya taps "Cancel" before confirming, then she returns to FEAT-01.SPEC-004 and no kid profile is created.

**FEAT-01.SPEC-007-AC-05:** Given Maya has already added one kid profile in this session, when she starts adding a second, then this screen shows again from its Unconfirmed state, requiring a fresh confirmation.

**FEAT-01.SPEC-007-AC-06:** Given Maya checks the box and then unchecks it, when she looks at "Continue", then it is disabled again.

**FEAT-01.SPEC-007-AC-07:** Given Maya taps "Continue" twice in rapid succession after confirming, then only one navigation to FEAT-01.SPEC-005 occurs.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 3 (unconfirmed, confirmed, offline N/A) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



# Screen Spec: Weekly Budget & Schedule Setup

## Overview

**Name:** Weekly Budget & Schedule Setup
**ID:** FEAT-01.SPEC-008
**Type:** Screen
**Purpose:** The organiser states the weekly food budget and marks which nights are short on time.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Setting the household's weekly_budget in its configured currency
- Marking which nights of the week are time-constrained and the time limit for those nights
- Editing both later from FEAT-01.SPEC-010

**Non-Goals:**
- Setting the household's currency or unit system -- owned by FEAT-16 (Units, Currency & Locale Configuration), reached elsewhere in the guided setup journey; this screen only enters an amount in whatever currency is already configured
- Generating or evaluating a plan against the budget -- owned by FEAT-03 (AI Weekly Dinner Plan Generation) and FEAT-23 (Manual Weekly Planning), which read this data
- Per-night meal planning of any kind -- this screen only records which nights are time-constrained, not what is cooked on them

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser continues from adding members | Guided-setup wizard context |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser taps "Budget & Schedule" | Existing weekly_budget and weekly_schedule pre-filled; screen behaves in edit mode |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Set and edit the weekly budget and schedule | -- |
| Sam (Other Adult Member) | No | No | This screen is not exposed to Sam; the resulting budget and schedule facts are visible to him read-only through FEAT-01.SPEC-010 |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1) | No | Riley never reaches this screen directly; budget and schedule facts are visible only inside the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Entered values are preserved locally per FEAT-01.SPEC-013 and restored after re-authentication |

## Layout and Content

**Header (guided setup):** Wizard shell, "Step 5 of 8," title "Budget and busy nights." **Header (standalone, from Settings Hub):** Title "Budget & Schedule," back arrow to FEAT-01.SPEC-010.

**Body:** Two sections.
- **Weekly budget:** A single numeric field labeled "Roughly how much do you want to spend on groceries each week?" showing the household's configured currency symbol, with helper text: "This is a rough guide, not a hard limit -- if no safe week fits, you'll see the closest option and how much it goes over."
- **Weekly schedule:** Seven toggles, one per day of the week (Monday through Sunday), each labeled with the day name and, when toggled on, a time-limit selector (e.g., "30 minutes," "45 minutes") appears beside it. Helper text above the toggles: "Mark any nights that are short on time -- we'll only suggest quick dinners for those."

**Footer:** "Continue" button (guided setup) or "Save" button (edit mode), plus "Cancel" link in edit mode.

### Responsive Behavior

- **Compact breakpoint:** Budget field full width; day toggles stack one per row, each with its time-limit selector beside it.
- **Medium size class and above:** Content caps at the platform-wide form width, horizontally centered; day toggles render in a compact grid of two columns.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Weekly budget field | Type | Captures the amount; draft saved locally per FEAT-01.SPEC-013 | Field shows entered value | Standard input focus state |
| Weekly budget field | Blur | Validates via FEAT-01.SPEC-014 (positive amount) | Error state if invalid | "Enter an amount greater than zero" below the field |
| Day toggle | Tap | Marks/unmarks that day as time-constrained | Toggle state changes; time-limit selector appears/disappears | Immediate |
| Time-limit selector (per toggled day) | Select | Sets the time limit for that day | Selector shows chosen limit | Selected value displayed |
| "Continue" button (guided setup) | Tap | 1. Validate the budget. 2. Save weekly_budget and weekly_schedule to the Household. | Inline saving confirmation | Success: navigates to FEAT-16.SPEC-001 (Units & Currency Settings), guided setup's next step. Failure: inline error, entered data retained |
| "Save" button (edit mode) | Tap | Validate and update the Household's weekly_budget and weekly_schedule fields | Inline saving confirmation | Success: toast "Budget and schedule updated," returns to FEAT-01.SPEC-010. Failure: inline error, entered data retained |
| "Cancel" link (edit mode) | Tap | Discards the in-progress edit | Screen closes | Returns to FEAT-01.SPEC-010 without saving |

### Accessibility Notes

- **Focus order:** Weekly budget field -> Day toggles in day order (each followed immediately by its own time-limit selector when visible) -> Continue/Save -> Cancel (edit mode only).
- **Validation announcements:** The budget error is announced and associated with its field.
- **Dynamic field announcement:** When a day's time-limit selector appears or disappears (toggle on/off), the change is announced so the appearing control is not missed.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (guided setup, default) | Budget field empty, no days toggled | First-time arrival at this step | User enters a budget or toggles a day |
| Filling | Budget entered and/or one or more days toggled | User interacts with any field | User taps Continue/Save or navigates away |
| Saving | Inline saving confirmation, fields disabled briefly | User taps Continue/Save with valid input | Save completes or fails |
| Error | Failed field highlighted with error message; entered data retained | Validation fails or save fails | User corrects and retries |
| Edit (pre-filled) | Budget and schedule pre-filled with current values | Arrived from FEAT-01.SPEC-010 | User saves, cancels, or navigates away |
| Offline/Degraded | Banner: "You're offline -- this will save when you reconnect." Fields remain editable; Continue/Save queues the change locally per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored -- queued save submits automatically |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules). See that spec for the budget positivity rule. Checked on field blur and on submission. The weekly schedule carries no validation beyond data type -- an empty schedule (no days marked) is a valid, deliberate statement of "no time constraint," per this feature's Primary Flows & Alternates.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful save (guided setup) | FEAT-16.SPEC-001 (Units & Currency Settings) | FEAT-16 (Units, Currency & Locale Configuration) |
| Successful save (edit mode) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |
| Cancel (edit mode) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |

## Data Model

**Creates:** None.
**Reads:** In edit mode, Household.weekly_budget and Household.weekly_schedule.
**Updates:** Household -- weekly_budget (a positive amount in the household's configured currency) and weekly_schedule (which nights are time-constrained and their time limit).
**Deletes:** None.

## Business Rules

- An absent schedule (no days marked) means no time constraint applies to any night -- not a hard block on proceeding, per this feature's Primary Flows & Alternates: "Partial setup."
- The weekly budget is a rough guide, not a hard limit; how a plan behaves when no safe week fits the budget is defined by FEAT-03, not this screen.
- Later edits to budget or schedule from FEAT-01.SPEC-010 apply to the next plan, not retroactively, per this feature's Primary Flows & Alternates.

## Edge Cases

- **Organiser leaves the budget field empty and taps Continue** -- Since weekly_budget is optional during partial setup (per this feature's Validation & Limits), Continue proceeds with no budget set rather than blocking; only a non-empty value must be positive.
- **Organiser enters a zero or negative budget** -- Error: "Enter an amount greater than zero"; the field is not saved until corrected.
- **Organiser toggles a day on, then off, without selecting a time limit** -- No time-constraint entry is saved for that day; toggling off discards any partially selected limit for it.
- **Organiser edits the schedule from the Settings Hub while offline** -- The change queues locally and saves automatically once connectivity returns, per FEAT-01.SPEC-013.
- **Concurrent edit from two devices (Maya on laptop and phone)** -- Last-write-wins per the dependency map's Contention note for Household; a failed save on either device keeps the household's prior budget/schedule active rather than leaving a mixed state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (inbound) | Guided setup continues here |
| FEAT-16.SPEC-001 (Units & Currency Settings) | Navigation (outbound) | Next guided-setup step (units, currency, and aisle layout, FEAT-16, before FEAT-01.SPEC-009) |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Edit-mode entry and return |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Draft saving and offline queuing |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Budget positivity rule |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated access to this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| budget_set | amount provided (yes / no) | Household's weekly_budget is saved | supports success-metrics.md: "First-Session Onboarding Completion" |
| schedule_set | count of time-constrained nights | Household's weekly_schedule is saved | supports success-metrics.md: "First-Session Onboarding Completion" |

## Acceptance Criteria

**FEAT-01.SPEC-008-AC-01:** Given Maya is on this screen during guided setup, when she enters a weekly budget and toggles Monday and Wednesday as 30-minute nights and taps "Continue", then the Household's weekly_budget and weekly_schedule are saved and she is taken to FEAT-16.SPEC-001 (Units & Currency Settings), guided setup's next step.

**FEAT-01.SPEC-008-AC-02:** Given Maya enters "0" as her weekly budget, when she blurs the field, then the error "Enter an amount greater than zero" appears.

**FEAT-01.SPEC-008-AC-03:** Given Maya leaves the budget field empty and marks no nights, when she taps "Continue", then setup proceeds with no budget or schedule set, per the partial-setup allowance.

**FEAT-01.SPEC-008-AC-04:** Given Maya opens this screen from FEAT-01.SPEC-010 to change the budget, when she updates the amount and taps "Save", then the household's budget updates and she sees "Budget and schedule updated," returning to the Settings Hub.

**FEAT-01.SPEC-008-AC-05:** Given Maya toggles Friday on and then off again without selecting a time limit, when she saves, then Friday is not recorded as time-constrained.

**FEAT-01.SPEC-008-AC-06:** Given Maya is in edit mode with unsaved changes, when she taps "Cancel", then she returns to FEAT-01.SPEC-010 without the changes being saved.

**FEAT-01.SPEC-008-AC-07:** Given Maya loses connectivity while entering the budget, when she taps "Continue", then the banner "You're offline -- this will save when you reconnect." appears and the entry is queued locally.

**FEAT-01.SPEC-008-AC-08:** Given Maya is signed in on two devices and saves different budgets from each within moments, then the most recently saved value is the one that persists, per last-write-wins.

**FEAT-01.SPEC-008-AC-09:** Given Maya sets a weekly schedule marking three weeknights as time-constrained, then FEAT-03/FEAT-23 read that schedule when proposing dinners for those nights (verified by the schedule_set event and the saved weekly_schedule field).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (empty, filling, saving, error, edit, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Setup Complete & Next Steps

## Overview

**Name:** Setup Complete & Next Steps
**ID:** FEAT-01.SPEC-009
**Type:** Screen
**Purpose:** The organiser sees setup is complete and chooses between manual planning (free) or upgrading (paid).
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Confirming guided setup is complete
- Branching the organiser to manual planning (free tier) or the paid-tier overview, based on the choice they make here
- The terminal screen of the guided setup wizard

**Non-Goals:**
- Generating an AI plan -- owned by FEAT-03 (AI Weekly Dinner Plan Generation); this screen only routes the organiser toward the paid-tier overview (FEAT-14), it never generates a plan itself
- Building the manual week -- owned by FEAT-23 (Manual Weekly Planning); this screen only hands off to it
- Upgrading the subscription itself -- owned by FEAT-14 (Subscription & Billing Management); this screen only offers the entry point

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-16.SPEC-002 (Aisle Name Customization) | Organiser taps "Finish" after confirming units, currency, and aisle layout -- guided setup's step 7 | Guided-setup wizard context (final step) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Choose manual planning or upgrade | -- |
| Sam (Other Adult Member) | No | No | Sam never reaches this screen -- guided setup and its completion are the organiser's own flow; Sam's first view of the household is through FEAT-01.SPEC-010 once he is invited |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | No | No | N/A -- this is a one-time completion screen with no ongoing state; not exposed through operator support access |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Since the prior step already saved, the organiser resumes here directly after re-authenticating |

## Layout and Content

**Header:** Wizard shell, "Step 8 of 8," title "You're all set."

**Body:** A confirmation message: "{Household name} is ready." Below it, two choices presented as cards:
- **"Pick this week's dinners"** (shown to every household, since every household starts on the free tier): "Build this week's plan yourself from our recipe library -- free, always." A "Start planning" button.
- **"Upgrade for an AI-generated plan"**: "Get a full week proposed for you automatically, using your household's rules and budget." An "See plans" button.

**Footer:** None -- the two cards carry the terminal actions.

### Responsive Behavior

- **Compact breakpoint:** Cards stack full width, one above the other.
- **Medium size class and above:** Cards render side by side in a two-column layout, equal width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Start planning" button | Tap | Navigate to FEAT-23 (Manual Weekly Planning) for the current empty week | Screen changes | Standard transition, leaving this feature |
| "See plans" button | Tap | Navigate to FEAT-14 (Subscription & Billing Management) paid-tier overview | Screen changes | Standard transition, leaving this feature |

### Accessibility Notes

- **Focus order:** Confirmation message (read order) -> "Start planning" card -> "Upgrade" card.
- **Completion announcement:** The "{Household name} is ready" confirmation is announced when the screen loads, since it marks the end of a multi-step flow.
- **Keyboard alternatives:** Both actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | The confirmation message renders as "{brief inline placeholder} is ready" -- only the household-name portion is placeholder text; both choice cards render immediately, fully interactive, since neither card's action depends on the household name resolving | Screen first opens, before the `Household.household_name` read (see Data Model) resolves | The name resolves (typically under a second, since it was already saved at FEAT-01.SPEC-003 earlier in the same guided-setup session) and the placeholder is replaced with the actual name, entering Complete |
| Complete (default and only steady state) | Confirmation message and both choice cards shown | The household-name read resolves | User chooses either card |
| Error | Confirmation message falls back to "Your household is ready" (name omitted) with a small inline "Retry" control next to it; both choice cards remain fully visible and interactive regardless, since this is the terminal screen of guided setup and neither card's action depends on the household name resolving | The `Household.household_name` read fails (e.g., a transient network or session issue) | Organiser taps "Retry" and the read succeeds, replacing the fallback text with the actual name; or the organiser proceeds via either card without waiting for the name to resolve |
| Offline/Degraded | Both cards remain visible; "Start planning" leads to FEAT-23, which is fully viewable offline per its own spec; "See plans" is disabled with "Upgrading needs a connection -- try again once you're back online." If the household name has not yet resolved when connectivity is lost, the Error fallback text is shown (offline is treated as a load failure for this single read, since there is nothing to retry until connectivity returns) | Connectivity lost while this screen is open | Connectivity restored -- "See plans" re-enables, and the household name resolves if it had not already |

## Validation Rules

**Option B -- Inline (no user input on this screen):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| N/A | This screen collects no input | -- | N/A -- no validation applies; the screen presents two navigation choices only |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Start planning" tap | -- | FEAT-23 (Manual Weekly Planning) |
| "See plans" tap | FEAT-14.SPEC-001 (Plan Tier Overview) | FEAT-14 (Subscription & Billing Management) |

## Data Model

**Creates:** None.
**Reads:** Household.household_name, for the confirmation message. This read can be slow or fail like any data fetch (see States: Loading, Error), even though the value was already saved earlier in the same guided-setup session at FEAT-01.SPEC-003 -- the read still crosses the network on this screen's own load, since guided setup does not carry the name forward in client-side memory between steps.
**Updates:** None -- this screen marks the end of the guided setup flow conceptually, but no separate "setup_complete" field exists on the Household beyond having all its prior steps' data already saved.
**Deletes:** None.

## Business Rules

- Every new household starts on the free tier (FEAT-01.SPEC-011 provisions this automatically), so "Pick this week's dinners" is always available regardless of which choice the organiser makes here.
- Choosing "See plans" does not itself generate a plan or complete a purchase -- it only opens the paid-tier overview (FEAT-14), consistent with this feature's Communications field: "on the paid tier, the first AI plan is on its way" only after the organiser actually subscribes.
- This screen never promises AI plan generation to a free-tier household, per this feature's synthesis correction (product-features.md, Communications): the free tier's path is manual planning, not a generated first plan.

## Edge Cases

- **Organiser closes the app on this screen without choosing either card** -- No data is lost; the household and all prior setup data are already saved. Returning later (via FEAT-01.SPEC-010, since a household now exists) shows the Settings Hub rather than this screen again, since guided setup does not re-trigger for a completed household.
- **Organiser taps both cards in quick succession** -- The first tap's navigation takes precedence; the second tap is ignored once navigation has begun.
- **Organiser is offline and taps "See plans"** -- The button is disabled with the message stating a connection is needed; "Start planning" remains available since FEAT-23 supports offline viewing.
- **Household referral attribution (FEAT-24) is still pending when this screen loads** -- Attribution completed at household creation (FEAT-01.SPEC-003), so this screen has no dependency on it and displays normally regardless.
- **The `Household.household_name` read fails or is slow** -- Both choice cards render immediately regardless, since neither depends on the name; the confirmation message shows a brief placeholder while loading, then falls back to "Your household is ready" with a "Retry" control if the read fails outright. The organiser is never blocked from choosing either card while waiting on or recovering from this read.
- **Organiser taps "Retry" on the fallback message multiple times in quick succession** -- Only one read is in flight at a time; extra taps while a retry is already pending are ignored until it resolves.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-002 (Aisle Name Customization) | Navigation (inbound) | Final guided-setup step (FEAT-16, units/currency/aisles) arrives here after FEAT-01.SPEC-008 |
| FEAT-23 (Manual Weekly Planning) | Navigation (outbound) | "Start planning" hands off here |
| FEAT-14 (Subscription & Billing Management) | Navigation (outbound) | "See plans" hands off here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| setup_completed | choice made (manual / upgrade) | Screen loads (guided setup's final step reached) | supports success-metrics.md: "First-Session Onboarding Completion" |

## Acceptance Criteria

**FEAT-01.SPEC-009-AC-01:** Given Maya completes guided setup's final step, FEAT-16.SPEC-002 (Aisle Name Customization), and taps "Finish", when this screen loads, then she sees "{Household name} is ready" along with the "Pick this week's dinners" and "Upgrade for an AI-generated plan" cards.

**FEAT-01.SPEC-009-AC-02:** Given Maya is on this screen, when she taps "Start planning", then she is taken to Manual Weekly Planning (FEAT-23) for the current empty week.

**FEAT-01.SPEC-009-AC-03:** Given Maya is on this screen, when she taps "See plans", then she is taken to the Subscription & Billing Management (FEAT-14) paid-tier overview, and no plan is generated and no purchase completes automatically.

**FEAT-01.SPEC-009-AC-04:** Given Maya closes the app on this screen without choosing either card, when she reopens the product later, then she lands on FEAT-01.SPEC-010 (Household Settings Hub), not back on this screen, since her household is already fully set up.

**FEAT-01.SPEC-009-AC-05:** Given Maya is offline on this screen, when she looks at "See plans", then it is disabled with "Upgrading needs a connection -- try again once you're back online."

**FEAT-01.SPEC-009-AC-06:** Given Maya is offline on this screen, when she taps "Start planning", then she is still taken to FEAT-23, which supports offline viewing.

**FEAT-01.SPEC-009-AC-07:** Given Maya taps "Start planning" and "See plans" in rapid succession, then only the first tap's navigation occurs.

**FEAT-01.SPEC-009-AC-08:** Given Maya reaches this screen, when the `Household.household_name` read has not yet resolved, then she sees both choice cards fully interactive immediately while the confirmation message shows a brief placeholder in place of the household name.

**FEAT-01.SPEC-009-AC-09:** Given Maya reaches this screen and the `Household.household_name` read fails, when the failure occurs, then she sees "Your household is ready" with a "Retry" control, both choice cards remain fully interactive, and tapping "Retry" re-attempts the read and replaces the fallback text with the actual name on success.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 2 | 2 |
| States | 4 (loading, complete, error, offline) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Screen Spec: Household Settings Hub

## Overview

**Name:** Household Settings Hub
**ID:** FEAT-01.SPEC-010
**Type:** Screen
**Purpose:** The organiser revisits and edits any setup area later; other adult members view household facts and link out to their own preferences.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Displaying the household's current facts: name, member count, budget, schedule, plan-arrival day/time
- Re-opening every setup screen in edit mode (household name, members, budget & schedule)
- Editing the plan-arrival day and time inline on this hub (organiser only), governed by FEAT-07.SPEC-004
- Linking out to units/currency/locale (FEAT-16), notification preferences (FEAT-07, FEAT-13), support access record (FEAT-22), and calendar connection (FEAT-21, Later)
- Linking out to invitation management (FEAT-09.SPEC-001), organiser hand-over initiation (FEAT-09.SPEC-003), a pending hand-over request addressed to the signed-in adult (FEAT-09.SPEC-004), and leaving the household (FEAT-09.SPEC-005)
- Read-only display of household facts for Sam, plus his own link to his own preferences

**Non-Goals:**
- Editing units, currency, or aisle names directly -- owned by FEAT-16; this hub only links out to it
- Performing invitation sending/revoking, the organiser hand-over transfer itself, or member removal -- owned by FEAT-09 and FEAT-18; this hub only links to those screens, which own the actions and their own authorization checks
- Billing management -- owned by FEAT-14; not exposed from this hub at all, since Billing access is None for everyone but the organiser through FEAT-14's own screens

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Existing organiser or adult member signs in to an account with a household | None |
| FEAT-01.SPEC-002 (Password Recovery) | Reset completes for an account with a household | None |
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | Organiser returns to the app after completing guided setup without choosing a next step | None |
| Any screen | Organiser or Sam navigates to "Household settings" from the product's persistent navigation | None |
| FEAT-09.SPEC-012 (Invitation Accepted Confirmation) | Organiser taps "View household" | None |
| FEAT-09.SPEC-013 (Member Left Household Notification) | Organiser taps "View household" | None |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | New organiser taps "Go to Household Settings" on the Success state, or the recipient confirms a decline, or taps "Back to Household" on the withdrawn state | None -- the hub loads with the viewer's current role (organiser access after a completed hand-over) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen, all facts and edit entry points | Edit household name, members, budget & schedule, plan-arrival day/time; open all linked-out settings; manage invitations (FEAT-09.SPEC-001); hand over the organiser role (FEAT-09.SPEC-003) | -- |
| Sam (Other Adult Member) | Full screen, all household facts (read-only) | View only for household-owned facts including plan-arrival day/time; can open his own notification preferences (FEAT-07, FEAT-13) to edit them; can act on a hand-over request addressed to him (FEAT-09.SPEC-004) when one is pending; can leave the household (FEAT-09.SPEC-005) | Edit entry points for household name, members, budget, schedule, and plan-arrival are not shown to Sam; a direct navigation attempt to one shows "Only the organiser can change this" and returns here. "Household Invitations" and "Hand over organiser role" are not shown to Sam; a direct navigation attempt shows the destination screen's own denial message and returns here. |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1) | No | Riley never reaches this screen directly; equivalent household facts are visible only inside the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." No in-progress edit exists on this hub screen itself to preserve |

## Layout and Content

**Header:** Title "Household settings," with the household name as a subtitle.

**Body:** A vertical list of setting rows, each showing its current summary value and (organiser only) a chevron indicating it opens an edit screen:
- "Household name" -- current name; opens FEAT-01.SPEC-003 in edit mode (organiser only)
- "Members" -- member count; opens FEAT-01.SPEC-004 (organiser: full access; Sam: read-only)
- "Budget & schedule" -- current budget and count of time-constrained nights; opens FEAT-01.SPEC-008 in edit mode (organiser only)
- "Plan arrival" -- current day and time-slot summary (e.g., "Sunday" with the chosen Evening slot from platform parameter: `plan-arrival-time-slots`); tapping the row (organiser only) expands an inline day/time picker in place, governed by FEAT-07.SPEC-004; Sam sees the same summary as a plain read-only fact, no chevron
- "Units, currency & aisles" -- links to FEAT-16 (organiser only, per that feature's own access rules)
- "Notification preferences" -- opens the signed-in adult's own FEAT-01.SPEC-005 (Member Profile Detail) at its plan-ready and nightly-nudge toggles, whose rules are owned by FEAT-07 and FEAT-13 (every adult, own preferences only)
- "Household Invitations" -- opens FEAT-09.SPEC-001 (organiser only); absent from Sam's list entirely
- "Hand over organiser role" -- opens FEAT-09.SPEC-003 (organiser only); absent from Sam's list entirely
- Pending hand-over entry -- shown only to an Other Adult Member who currently has a hand-over request addressed to them: "{organiser display name} wants to make you the organiser" opens FEAT-09.SPEC-004; absent for everyone else, including the organiser herself, since only one such request can exist at a time (XBR-15) and it is addressed to exactly one recipient
- "Leave household" -- opens FEAT-09.SPEC-005 (Other Adult Member only); absent from the organiser's list entirely, per FEAT-09.SPEC-005's XBR-15 gate
- "Support access record" -- links to FEAT-22's record of when and why support viewed the household (organiser: full view; not shown to Sam, per the Access Matrix's Support View column)
- "Connect calendar" (Later phase) -- links to FEAT-21 (organiser only)

For Sam, rows without an organiser-only edit path render as plain informational rows (no chevron); "Notification preferences," his pending hand-over entry (when one is addressed to him), and "Leave household" remain live links to their own destinations.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Rows stack full width, one per row.
- **Medium size class and above:** Rows remain single-column but cap at a consistent platform-wide content width, horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Household name" row (organiser only) | Tap | Navigate to FEAT-01.SPEC-003 in edit mode | Screen changes | Standard transition |
| "Members" row | Tap | Navigate to FEAT-01.SPEC-004 | Screen changes | Standard transition; organiser sees full access, Sam sees read-only |
| "Budget & schedule" row (organiser only) | Tap | Navigate to FEAT-01.SPEC-008 in edit mode | Screen changes | Standard transition |
| "Plan arrival" row (organiser only) | Tap | Expands an inline day/time picker (day selector + time-slot selector, values from platform parameter: `plan-arrival-time-slots`) | Row expands in place | Picker appears inline, focus moves to the day selector |
| Day selector (plan-arrival picker, organiser only) | Select | Sets the pending day component of plan_arrival_day_time | Selector shows chosen day | Immediate |
| Time-slot selector (plan-arrival picker, organiser only) | Select | Sets the pending time-slot component of plan_arrival_day_time | Selector shows chosen slot | Immediate |
| "Save" button (plan-arrival picker, organiser only) | Tap | Validate and save plan_arrival_day_time per FEAT-07.SPEC-004 (day and time-slot must both be present) | Inline saving confirmation; row collapses back to summary on success | Success: toast "Plan arrival updated." Failure: inline error ("Choose both a day and a time for your plan to arrive." when the pair is incomplete), picker stays open with the pending selection retained |
| "Units, currency & aisles" row (organiser only) | Tap | Navigate to FEAT-16 | Screen changes | Standard transition, leaving this feature |
| "Notification preferences" row | Tap | Navigate to the signed-in member's own FEAT-01.SPEC-005 (Member Profile Detail), Notification preferences toggles (rules owned by FEAT-07/FEAT-13) | Screen changes | Standard transition |
| "Household Invitations" row (organiser only) | Tap | Navigate to FEAT-09.SPEC-001 | Screen changes | Standard transition, leaving this feature |
| "Hand over organiser role" row (organiser only) | Tap | Navigate to FEAT-09.SPEC-003 | Screen changes | Standard transition, leaving this feature |
| Pending hand-over entry (Other Adult Member with an addressed request) | Tap | Navigate to FEAT-09.SPEC-004 | Screen changes | Standard transition, leaving this feature |
| "Leave household" row (Other Adult Member only) | Tap | Navigate to FEAT-09.SPEC-005 | Screen changes | Standard transition, leaving this feature |
| "Support access record" row (organiser only) | Tap | Navigate to FEAT-22's support-visit record | Screen changes | Standard transition, leaving this feature |
| "Connect calendar" row (organiser only, Later) | Tap | Navigate to FEAT-21 | Screen changes | Standard transition, leaving this feature |

### Accessibility Notes

- **Focus order:** Rows in the order listed above, top to bottom; when the plan-arrival picker is expanded, focus order continues into the day selector, then the time-slot selector, then Save, before resuming the remaining rows.
- **Role-based visibility announcement:** Rows not shown to Sam are simply absent from the page structure (not present-but-hidden), so no announcement of a missing control is needed. The pending hand-over entry appears and disappears the same way -- absent from the structure, not hidden -- so it needs no separate "new item" announcement beyond the normal screen-load read-out.
- **Picker expand/collapse announcement:** The plan-arrival picker expanding or collapsing, and the "Plan arrival updated" confirmation, are announced.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | All applicable rows shown with current summary values | Screen opens and household data loads | Always the state once loaded |
| Loading | Rows render with a brief inline placeholder | Screen first opens | Data loads (typically under a second) |
| Error | Banner: "Couldn't load your household settings. Try again." with retry | Household data fails to load | Retry succeeds |
| Offline/Degraded | Previously loaded summary values remain fully viewable; edit entry points remain reachable, leading to screens that queue their own saves per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored |

## Validation Rules

**Option B -- Inline (the plan-arrival picker is this screen's only direct input; every other row displays summaries and links only):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| plan_arrival_day_time (day + time-slot pair) | Both a day and a time-slot must be present together; allowed values governed by FEAT-07.SPEC-004 (platform parameter: `plan-arrival-time-slots`) | On Save, in the plan-arrival picker | "Choose both a day and a time for your plan to arrive." |
| All other rows | No direct input; all editing happens on the destination screens | -- | N/A -- no validation applies here |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Household name" row tap | FEAT-01.SPEC-003 (Household Naming & Guided Setup Start, edit mode) | -- |
| "Members" row tap | FEAT-01.SPEC-004 (Member List & Add Member) | -- |
| "Budget & schedule" row tap | FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup, edit mode) | -- |
| "Plan arrival" row tap, Save | -- (inline picker on this screen; no navigation) | -- |
| "Units, currency & aisles" row tap | FEAT-16.SPEC-001 (Units & Currency Settings) | FEAT-16 (Units, Currency & Locale Configuration) |
| "Notification preferences" row tap | FEAT-01.SPEC-005 (Member Profile Detail, own profile, Notification preferences toggles) | -- (rules owned by FEAT-07 / FEAT-13) |
| "Household Invitations" row tap | FEAT-09.SPEC-001 (Household Invitations Manager) | FEAT-09 (Household Invitations & Membership) |
| "Hand over organiser role" row tap | FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | FEAT-09 (Household Invitations & Membership) |
| Pending hand-over entry tap | FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | FEAT-09 (Household Invitations & Membership) |
| "Leave household" row tap | FEAT-09.SPEC-005 (Leave Household) | FEAT-09 (Household Invitations & Membership) |
| "Support access record" row tap | FEAT-22.SPEC-003 (Household Support Access Record) | FEAT-22 (Operator Read-Only Support Access) |
| "Connect calendar" row tap (Later) | FEAT-21.SPEC-001 (Calendar Connection Settings) | FEAT-21 (Family Calendar Sync) |

## Data Model

**Creates:** None.
**Reads:** Household -- household_name, weekly_budget, weekly_schedule (summarized), plan_arrival_day_time; Household.pending_organiser_handover (the workflow marker defined in FEAT-09.SPEC-003, read to determine whether a pending hand-over entry addressed to the signed-in Other Adult Member should render); Member Profile -- count of Active members; Support Request -- access_record summary, for the support-access row.
**Updates:** Household -- plan_arrival_day_time (organiser only, per FEAT-07.SPEC-004). All other edits happen on the destination screens this hub links to.
**Deletes:** None.

## Business Rules

- Only the organiser sees edit entry points for household name, members (add/invite), budget & schedule, and plan-arrival day/time, per FEAT-01.SPEC-016.
- Every adult sees and controls only their own notification preferences from this hub, per XBR-13.
- The support-access record shown here is the organiser's View-level visibility into FEAT-22's read-only support access, per XBR-14 -- Maya can see when and why Riley viewed the household, but this hub never grants her any control over that access itself.
- The allowed values, default, and save validation for plan_arrival_day_time are governed by FEAT-07.SPEC-004; this hub enforces them inline but does not define them, per that spec's Business Rules ("FEAT-01.SPEC-010's edit screen ... defer[s] to it rather than duplicating the value rules").
- "Household Invitations," "Hand over organiser role," and "Leave household" link to FEAT-09's own screens, which own the underlying actions and their own authorization checks (FEAT-09.SPEC-011); this hub's row-visibility rules mirror those checks so a control is never shown that the destination screen would itself reject.
- A pending hand-over entry appears only for the specific Other Adult Member the outstanding request names (Household.pending_organiser_handover); no other member, and never the organiser herself, sees it, per XBR-15's single-outstanding-request rule.

## Edge Cases

- **Sam attempts to reach an organiser-only edit screen via direct navigation** -- Shown "Only the organiser can change this" and returned to this hub.
- **Household data fails to load** -- Error banner with retry; no stale or partial summary values are shown in place of failed data.
- **Organiser edits budget from another device while this hub is open** -- The summary value updates to reflect the change without requiring a manual refresh, consistent with this entity's low-contention profile; a failed save on the other device leaves this hub's displayed value unchanged.
- **Calendar connection (Later phase) is not yet available for a v1 household** -- The "Connect calendar" row is simply absent until FEAT-21 ships; this is a phase gate, not a permission gate.
- **Organiser is offline and taps an edit row** -- Navigation still proceeds to the destination screen, which itself surfaces the offline state per its own spec.
- **Organiser saves an incomplete plan-arrival pair (a day selected but no time-slot, or vice versa)** -- Save is blocked with "Choose both a day and a time for your plan to arrive."; the hub's displayed summary remains the previous complete value.
- **Organiser changes plan-arrival day/time from another device while this hub is open** -- The summary value updates to reflect the change without requiring a manual refresh, consistent with the Household entity's low-contention profile (the same pattern as the budget summary); a failed save on the other device leaves this hub's displayed value unchanged.
- **A pending hand-over request addressed to Sam is cancelled, declined, or accepted while he has this hub open** -- The pending hand-over entry is removed from the hub without further action from him, since the underlying marker it reads has been cleared.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (inbound) | Returning organiser/adult lands here |
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Navigation (outbound) | Household name edit entry point |
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (outbound) | Members entry point |
| FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | Navigation (outbound) | Budget & schedule edit entry point |
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | Navigation (inbound) | Organiser returning without choosing a next step lands here |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated row visibility |
| FEAT-16 (Units, Currency & Locale Configuration) | Navigation (outbound) | Locale settings entry point |
| FEAT-07 (Weekly Plan Ready Notification) | Navigation (outbound) | Plan-ready preference entry point |
| FEAT-07.SPEC-004 (Plan-Arrival Day & Time Setting Rule) | References (inbound) | Governs allowed values, default, and save validation for the inline plan-arrival picker |
| FEAT-13 (Tonight's Dinner Reminder) | Navigation (outbound) | Nightly nudge preference entry point |
| FEAT-09.SPEC-001 (Household Invitations Manager) | Navigation (outbound) | Household Invitations entry point |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Navigation (outbound) | Hand-over-role entry point |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Navigation (outbound) | Pending hand-over entry point for the addressed recipient |
| FEAT-09.SPEC-005 (Leave Household) | Navigation (outbound) | Leave-household entry point |
| FEAT-22 (Operator Read-Only Support Access) | Navigation (outbound) | Support access record entry point |
| FEAT-21 (Family Calendar Sync) | Navigation (outbound) | Calendar connection entry point (Later) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| household_settings_opened | role of viewer | Screen opens | N/A -- no Stage 2 metric tracks settings visits; retained as an operational signal, not a first-session-onboarding step this metric measures |
| household_settings_edit_entry_selected | which row selected | Organiser taps an edit entry point | N/A -- the resulting edit itself is measured by the destination screen's own events (e.g., household_name_edited); this row-selection event is a UI-navigation signal only |
| plan_arrival_updated | new day/time-slot pair | Organiser saves a change in the plan-arrival picker | N/A -- no Stage 2 metric tracks this setting change directly; FEAT-07's own delivery-eligibility metrics measure the resulting plan-ready message, not this edit |

## Acceptance Criteria

**FEAT-01.SPEC-010-AC-01:** Given Maya signs in to an account with an existing household, when the sign-in succeeds, then she lands on this hub showing the household's name, budget, and schedule summaries.

**FEAT-01.SPEC-010-AC-02:** Given Maya taps "Household name", then she is taken to FEAT-01.SPEC-003 in edit mode, pre-filled with the current name.

**FEAT-01.SPEC-010-AC-03:** Given Sam opens this hub, when it loads, then he sees all household facts but no edit chevrons on household name, members, budget & schedule, or plan arrival, and no "Household Invitations" or "Hand over organiser role" rows at all.

**FEAT-01.SPEC-010-AC-04:** Given Sam attempts to navigate directly to the budget edit screen, then he sees "Only the organiser can change this" and is returned to this hub.

**FEAT-01.SPEC-010-AC-05:** Given Sam opens "Notification preferences" from this hub, then he reaches his own preference screen and can edit only his own settings.

**FEAT-01.SPEC-010-AC-06:** Given Maya opens "Support access record", then she is taken to FEAT-22's record showing when and why Riley viewed the household.

**FEAT-01.SPEC-010-AC-07:** Given the household's data fails to load, when this screen opens, then the banner "Couldn't load your household settings. Try again." appears with a retry option.

**FEAT-01.SPEC-010-AC-08:** Given Maya has this hub open on her phone and updates the budget from her laptop, when the laptop save succeeds, then the phone's displayed budget summary updates without a manual refresh.

**FEAT-01.SPEC-010-AC-09:** Given Maya is offline, when she taps "Household name", then she is still taken to FEAT-01.SPEC-003, which itself shows the offline state for the household-name edit.

**FEAT-01.SPEC-010-AC-10:** Given the product is running before FEAT-21 (Family Calendar Sync) has shipped, when this hub loads, then no "Connect calendar" row appears.

**FEAT-01.SPEC-010-AC-11:** Given Maya taps "Plan arrival", selects Wednesday and one of the available Morning slots, and taps "Save", then the household's plan_arrival_day_time updates to Wednesday paired with that slot, the toast "Plan arrival updated" appears, and the row collapses back to the new summary.

**FEAT-01.SPEC-010-AC-12:** Given Maya selects a day but no time-slot in the plan-arrival picker and taps "Save", then the error "Choose both a day and a time for your plan to arrive." appears, the picker stays open, and the household's previous plan_arrival_day_time value remains active.

**FEAT-01.SPEC-010-AC-13:** Given Sam opens this hub, when it loads, then he sees the plan-arrival day/time as a read-only summary with no chevron and no picker control.

**FEAT-01.SPEC-010-AC-14:** Given Maya taps "Household Invitations", then she is taken to FEAT-09.SPEC-001 (Household Invitations Manager).

**FEAT-01.SPEC-010-AC-15:** Given Maya taps "Hand over organiser role", then she is taken to FEAT-09.SPEC-003 (Organiser Hand-Over Initiation).

**FEAT-01.SPEC-010-AC-16:** Given Sam has a hand-over request currently addressed to him, when he opens this hub, then he sees the pending hand-over entry "Maya wants to make you the organiser" and tapping it takes him to FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance).

**FEAT-01.SPEC-010-AC-17:** Given Maya (Organiser) opens this hub, then no "Leave household" row and no pending hand-over entry appear for her, since both actions are exclusive to an Other Adult Member per FEAT-09.SPEC-005 and XBR-15.

**FEAT-01.SPEC-010-AC-18:** Given Sam taps "Leave household", then he is taken to FEAT-09.SPEC-005 (Leave Household).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 15 | 15 |
| States | 4 (loaded, loading, error, offline) | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 8 | 8 |



# Automation Spec: Default Subscription Provisioning

## Overview

**Name:** Default Subscription Provisioning
**ID:** FEAT-01.SPEC-011
**Type:** Automation
**Purpose:** When a Household is created, the system automatically provisions a free-tier Subscription record so the household is never left without one.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Creating exactly one Subscription record, defaulted to the free tier, at the moment a Household record is created
- Guaranteeing this happens as an inseparable part of household creation, never as a later or optional step

**Non-Goals:**
- Upgrading, downgrading, billing period changes, or cancellation -- all owned entirely by Subscription & Billing Management (FEAT-14); this automation only ever creates the initial free-tier record
- Payment collection of any kind -- the free tier requires no payment method, and this automation never interacts with the payment-processing capability
- Re-provisioning a Subscription for a household that already has one -- product-features.md's Entity-Lifecycle for Subscription defines exactly one Create operation per household, owned by this automation alone; no other path creates a second Subscription record

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household record created | FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Always, immediately after a new Household record is successfully saved | The new Household's identity, needed to attach the Subscription to the correct household |

## Processing Logic

1. Receive the newly created Household's identity from FEAT-01.SPEC-003.
2. Create a new Subscription record for that household.
3. Set the Subscription's tier to free, billing_period to none (no billing period applies to the free tier), and billing_state to Active.
4. Attach the Subscription to the Household so every feature that reads Subscription (FEAT-03, FEAT-05, FEAT-12, FEAT-24, FEAT-22) sees a valid record immediately.
5. Confirm the Subscription is readable before the household-creation flow (FEAT-01.SPEC-003) proceeds to its next screen, so no downstream screen ever observes a household with no Subscription.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Subscription provisioned | Household creation succeeds | New Subscription record created: tier=free, billing_state=Active | None -- this automation is silent; the organiser simply proceeds to FEAT-01.SPEC-004 as normal | FEAT-01.SPEC-003 (triggering spec), FEAT-01.SPEC-009 (reads tier when presenting the free/paid choice), FEAT-14 (owns all subsequent Subscription lifecycle) |
| Provisioning failure | The Subscription record cannot be created immediately after Household creation | Household record exists without a Subscription momentarily | Household creation itself is treated as not yet complete: FEAT-01.SPEC-003 shows its own error/retry state rather than proceeding, since this automation is a required, non-optional part of household creation | FEAT-01.SPEC-003 |

## Data Model

**Reads:** Household -- the newly created record's identity only.
**Creates:** Subscription -- tier, billing_period, billing_state fields, attached to the new Household.
**Updates:** None.
**Deletes:** None.

## Business Rules

- This automation runs synchronously as part of household creation (FEAT-01.SPEC-003) -- the triggering screen does not proceed to FEAT-01.SPEC-004 until provisioning succeeds, per XBR-05's expectation that every household has a valid Subscription from the moment it exists.
- Free-tier households generate no AI cost and require no payment method, consistent with scope-boundaries.md SC-16.
- This is the only creation path for Subscription; all other Subscription fields and states are owned exclusively by FEAT-14 (Subscription & Billing Management).

## Edge Cases

- **Household creation succeeds but Subscription provisioning fails** -- Treated as a single failed operation from the organiser's point of view: FEAT-01.SPEC-003 shows its retry-with-preserved-data error state, and household creation is retried as a whole rather than leaving an orphaned Household with no Subscription.
- **Organiser retries household creation after a provisioning failure** -- The retry re-attempts both household creation and Subscription provisioning together; no duplicate Household or Subscription is created from the failed attempt.
- **Concurrent trigger firing (Maya submits household creation from two devices at once)** -- Only one Household record and one attached Subscription result: the dependency map's Contention note for Household resolves this at the household-creation level (last-write-wins), and Subscription provisioning follows whichever Household record is created first; the second attempt is rejected as "a household already exists for this account" rather than creating a second Subscription.
- **Trigger fires while a previous run is in flight** -- Cannot occur for the same account: household creation itself is blocked from firing twice concurrently by FEAT-01.SPEC-003's own double-submit prevention, so this automation never has two in-flight runs for the same new household.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Triggered by (inbound) | Fires immediately on successful household creation |
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | Affects (outbound) | Reads the provisioned free tier when presenting the manual/upgrade choice |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Affects (outbound) | Reads Subscription for tier gating |
| FEAT-05 (Pantry-Aware Suggestions) | Affects (outbound) | Reads Subscription for tier-gated plan weighting |
| FEAT-12 (Meal Rating & Preference Learning) | Affects (outbound) | Reads Subscription for tier gating |
| FEAT-14 (Subscription & Billing Management) | Affects (outbound) | Owns all subsequent lifecycle of the provisioned record |

## Analytics and Success Signals

- **subscription_provisioned** (tier: free) -- N/A -- no Stage 2 metric measures provisioning itself; retained as an operational integrity signal confirming every household has a Subscription at creation.
- **subscription_provisioning_failed** (household reference) -- N/A -- diagnostic signal only; no Stage 2 metric tracks this failure mode, but its absence would silently violate XBR-05.

## Acceptance Criteria

**FEAT-01.SPEC-011-AC-01:** Given Maya completes FEAT-01.SPEC-003 and her Household record is created, when this automation fires, then a Subscription record is created for her household with tier set to free and billing_state set to Active.

**FEAT-01.SPEC-011-AC-02:** Given the Subscription is provisioned, when Maya reaches FEAT-01.SPEC-009, then the screen correctly shows her as a free-tier household with the "Pick this week's dinners" option available.

**FEAT-01.SPEC-011-AC-03:** Given Subscription provisioning fails immediately after Household creation, when FEAT-01.SPEC-003 detects this, then it shows its retry-with-preserved-data error state and does not proceed to FEAT-01.SPEC-004.

**FEAT-01.SPEC-011-AC-04:** Given Maya retries household creation after a provisioning failure, when the retry succeeds, then exactly one Household and one Subscription record exist -- not two of either.

**FEAT-01.SPEC-011-AC-05:** Given Maya's household is newly provisioned with a free-tier Subscription, when FEAT-05 (Pantry-Aware Suggestions) checks tier gating, then it correctly reads the household as free-tier (pantry logging available, plan-weighting not).

**FEAT-01.SPEC-011-AC-06:** Given Maya submits household creation from two devices at effectively the same time, when both attempts resolve, then exactly one Household and one attached Subscription exist, and the second attempt is told a household already exists for the account.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 2 (provisioned, failure) | 2 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



# Automation Spec: Mid-Week Hard-Rule Change Trigger

## Overview

**Name:** Mid-Week Hard-Rule Change Trigger
**ID:** FEAT-01.SPEC-012
**Type:** Automation
**Purpose:** A new or tightened hard dietary rule triggers an immediate re-check of the current week's plan, so a meal made unsafe by a rule change never stays on the plan.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Detecting when a saved Dietary Rule change is a new or tightened hard rule (an allergy or religious rule added, or a rule's strength or allergen scope widened)
- Handing off the current week's plan for re-checking to the Dietary Rules & Allergy Safety Engine (FEAT-02)
- Firing FEAT-01.SPEC-018 (Mid-Week Rule Change Notification) once the re-check identifies a removed meal

**Non-Goals:**
- Performing the safety re-check itself (which recipes fail, which pass) -- owned entirely by FEAT-02 (Dietary Rules & Allergy Safety Engine); this automation only hands off the request and reacts to its outcome
- Offering safe alternatives for a removed meal -- owned by One-Tap Meal Swap (FEAT-04), per XBR-02
- Updating the grocery list -- owned by Shared Grocery List (FEAT-06), which recalculates automatically once the plan changes, per XBR-03

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Hard dietary rule added | FEAT-01.SPEC-006 (Dietary Rules Editor) | The saved rule is an allergy or religious rule (rule_kind, strength=hard) that did not previously exist for this member | Member reference, rule_kind, allergen (if applicable), household reference |
| Hard dietary rule tightened | FEAT-01.SPEC-006 (Dietary Rules Editor) | An existing hard rule is edited to widen its scope (e.g., a broader allergen match, an added named ingredient making more recipes unsafe) | Member reference, the rule's previous and new values, household reference |

## Processing Logic

1. Receive the saved Dietary Rule change from FEAT-01.SPEC-006, including whether it is a creation or an edit and the rule's strength.
2. Determine whether the change qualifies as "new or tightened hard": a newly created allergy or religious rule always qualifies; an edited rule qualifies only if its scope widened (never on a softening edit, since a loosened rule cannot make an already-approved meal newly unsafe).
3. If the change does not qualify (e.g., a new soft dislike, or a hard rule edited to narrow its scope), take no further action -- this automation ends here silently.
4. If the change qualifies, identify the household's current week's Weekly Plan, if one exists.
5. If a current-week plan exists, request a re-check of its remaining (not-yet-cooked) Planned Meals against the household's full, updated Dietary Rule set from the Dietary Rules & Allergy Safety Engine (FEAT-02).
6. Receive the re-check's result: a list of Planned Meals that now fail the safety check (if any).
7. For each failing Planned Meal, request its removal from the plan (per FEAT-02's ownership of safety-driven removal) and its ingredients' removal from the current Grocery List (handled automatically by FEAT-06 once the plan changes, per XBR-03).
8. If one or more meals were removed, fire FEAT-01.SPEC-018 (Mid-Week Rule Change Notification) naming each removed meal.
9. If no current-week plan exists, or the re-check finds no failing meals, end silently -- there is nothing for the organiser to be told.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No current-week plan | Household has no active Weekly Plan for the current week | None | None | -- |
| Re-check finds no unsafe meals | A current-week plan exists but every remaining meal still passes the updated rule set | None | None -- the organiser is not told a re-check happened when nothing changed | FEAT-02 |
| One or more meals removed | The re-check finds one or more remaining meals that now violate the new/tightened rule | Affected Planned Meals removed from the current Weekly Plan; Grocery List recalculates | Organiser is told which meal(s) were removed, via FEAT-01.SPEC-018 | FEAT-02, FEAT-04 (offers alternatives), FEAT-06 (list recalculates), FEAT-01.SPEC-018 |
| Automation failure (re-check cannot complete) | The safety re-check itself cannot be completed against the current plan | No plan changes are made -- the automation fails closed, leaving the plan as it was rather than guessing | No immediate notification; the existing "checked against allergies" badges on the plan are treated as stale until a re-check succeeds, and the household's next scheduled interaction with the plan (e.g., opening it) re-attempts the check | FEAT-02, FEAT-01.SPEC-006 (the rule save itself still succeeds independently of the re-check's outcome) |

## Data Model

**Reads:** Dietary Rule -- the newly saved or edited rule, plus the household's full current rule set for the re-check; Weekly Plan and Planned Meal -- the current week's plan and its remaining meals.
**Creates:** None directly -- this automation orchestrates a hand-off; FEAT-02 and FEAT-04 own the entities they create or modify as a result.
**Updates:** Planned Meal -- status set to Removed (safety) for any meal the re-check fails, via FEAT-02's ownership.
**Deletes:** None directly.

## Business Rules

- This automation applies XBR-02 exactly: "A new or tightened hard rule takes effect on the current week immediately: remaining dinners are re-checked, any that now fail are flagged and removed, safe alternatives are offered through swap, the grocery list updates, and the organiser is told."
- Only hard rules (allergies and religious rules) trigger this automation -- a new dislike or a per-person vegetarian setting change never does, since dislikes are soft and never block a suggestion (per this feature's Dietary Rule Classification, FEAT-01.SPEC-015).
- Only the remaining (not-yet-cooked) portion of the current week's plan is re-checked -- meals already cooked are historical and are not retroactively altered.
- This automation never runs the safety determination itself; it is the trigger and hand-off point, while FEAT-02 owns the actual pass/fail logic per XBR-01.

## Edge Cases

- **Household has no current-week plan when the rule change is saved** -- The automation ends silently; there is nothing to re-check, and the new rule simply governs the next plan generated or built.
- **The tightened rule affects a meal already cooked earlier in the week** -- That meal is left untouched; only remaining, not-yet-cooked meals are in scope for re-check and removal.
- **Every remaining meal in the plan fails the re-check** -- Each is removed individually and each generates its own line in the FEAT-01.SPEC-018 notification; the household is left with an empty remainder of the week rather than a plan silently left inconsistent, and FEAT-04 offers alternatives for each open slot.
- **Concurrent trigger firing (two hard rules for different members saved within moments of each other)** -- Each triggers its own re-check independently against the household's rule set as it exists at that re-check's own start; a meal already removed by the first re-check is simply absent from the second re-check's remaining-meals list, so no duplicate removal or duplicate notification occurs for the same meal.
- **Trigger fires while a previous run is in flight** -- A second hard-rule change saved while an in-flight re-check has not yet completed queues behind it: the second re-check begins only after the first's removals (if any) are applied, ensuring it always evaluates the plan's true current state rather than a stale snapshot.
- **The dietary rule change is itself later reverted (e.g., an allergy entered in error and then removed) before the re-check completes** -- The in-flight re-check still completes against the rule set as it stood when the re-check began; if this produces a removal that the reversion would have avoided, the removed meal is not automatically restored (no restore path per this feature's Entity-Lifecycle Coverage Matrix) and the organiser must re-add it manually through FEAT-04 or FEAT-23.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-006 (Dietary Rules Editor) | Triggered by (inbound) | A new or tightened hard rule save fires this automation |
| FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules) | References (inbound) | Defines what counts as "hard" and "tightened" |
| FEAT-01.SPEC-018 (Mid-Week Rule Change Notification) | Triggers (outbound) | Fires when one or more meals are removed |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Triggers (outbound) | Performs the actual re-check and owns removal |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound) | Offers safe alternatives for any removed meal |
| FEAT-06 (Shared Grocery List) | Affects (outbound) | Recalculates automatically once the plan changes |

## Analytics and Success Signals

- **midweek_rule_change_recheck** (result: no_plan / no_unsafe_meals / meals_removed; removed_meal_count) -- supports success-metrics.md: "Zero Allergy Incidents" (this event is the concrete mechanism by which a tightened rule is enforced against an already-approved plan, directly serving the product's hard-zero allergy-safety promise).
- **midweek_recheck_failed** (household reference) -- N/A -- no Stage 2 metric tracks re-check failures directly, but this signal is retained because a silent failure here would otherwise undermine the Zero Allergy Incidents commitment without being observable.

## Acceptance Criteria

**FEAT-01.SPEC-012-AC-01:** Given Maya's household has a current-week plan with a Thursday dinner containing peanuts, when she adds a new peanut allergy for Jordan on FEAT-01.SPEC-006, then this automation fires, the re-check runs against the updated rule set, and Thursday's dinner is removed from the plan.

**FEAT-01.SPEC-012-AC-02:** Given a meal is removed by the re-check, then FEAT-01.SPEC-018 fires, telling Maya which meal was removed.

**FEAT-01.SPEC-012-AC-03:** Given Maya's household has no current-week plan, when she adds a new hard allergy, then this automation ends silently with no plan changes and no notification.

**FEAT-01.SPEC-012-AC-04:** Given Maya adds a new soft dislike (not a hard rule), when it is saved, then this automation does not fire at all.

**FEAT-01.SPEC-012-AC-05:** Given Maya edits an existing hard allergy to narrow its scope (a softening edit), when it is saved, then this automation does not fire.

**FEAT-01.SPEC-012-AC-06:** Given every remaining meal in the current week's plan fails the re-check after a new hard allergy is added, when the re-check completes, then each failing meal is individually removed and each is named in the resulting notification.

**FEAT-01.SPEC-012-AC-07:** Given two different hard rules for two different members are saved within moments of each other, when both trigger a re-check, then no meal is removed twice and no duplicate notification is sent for the same meal.

**FEAT-01.SPEC-012-AC-08:** Given the safety re-check cannot complete due to a processing failure, when this automation detects it, then no plan changes are made and the rule save itself still succeeds independently.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (new hard rule, tightened hard rule) | 2 |
| Outcome Paths | 4 (no plan, no unsafe meals, meals removed, automation failure) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Setup Draft Persistence & Offline Queuing

## Overview

**Name:** Setup Draft Persistence & Offline Queuing
**ID:** FEAT-01.SPEC-013
**Type:** Logic/Rule
**Purpose:** Governs how every setup screen saves drafts locally and resumes exactly where the organiser left off, whether online or offline.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles
**Governed Entity:** Setup draft state (a cross-cutting behavior, not itself a Shared Data Entity from the dependency map -- it governs in-progress, not-yet-confirmed input on Household, Member Profile, and Dietary Rule fields across FEAT-01.SPEC-003, 004, 005, 006, and 008)

## Scope and Non-Goals

**In Scope:**
- Saving in-progress form input locally as the organiser types, across every setup screen
- Resuming a setup screen exactly where it was left, after navigating away, closing the app, or losing connectivity
- Queuing a save made while offline and submitting it automatically once connectivity returns
- Failure behavior when a save cannot complete: preserving entered data and offering retry

**Non-Goals:**
- The offline behavior of screens outside this feature (e.g., Shared Grocery List's offline ticking) -- each feature owning offline-capable screens defines its own queuing behavior; this spec governs only FEAT-01's setup screens
- Conflict resolution between two devices editing the same confirmed (already-saved) record -- that is each screen's own concurrent-edit handling (e.g., FEAT-01.SPEC-003's Edge Cases), which is distinct from an in-progress, not-yet-saved draft
- Removing or expiring a draft that was never submitted -- product-features.md defines no automatic draft cleanup; a draft persists locally until it is either submitted or explicitly discarded by the organiser (e.g., tapping "Cancel" with confirmation)

## Governed Entity

**Entity:** Setup draft state (cross-cutting; applies to in-progress input on Household, Member Profile, and Dietary Rule fields)
**Source:** Feature Breakdown Brief, States field ("Offline-degraded: setup can be drafted offline and is held locally until connectivity returns to save") and Side-Effect Inventory ("Organiser closes the app mid-setup, with or without connectivity -> Draft is held locally; setup resumes exactly where it was left off when reopened")

| Field | Data Type | Description |
|-------|-----------|-------------|
| draft_screen | enum | Which setup screen the draft belongs to (FEAT-01.SPEC-003, 004, 005, 006, or 008) |
| draft_target | text | The specific record the draft applies to (e.g., which Member Profile, for FEAT-01.SPEC-005/006) |
| draft_fields | derived | The in-progress field values entered but not yet confirmed/saved |
| queued_operation | boolean | Whether this draft represents a save attempted while offline, awaiting submission |
| last_updated | date | When the draft was last modified, used to determine which of two devices' drafts is current if both exist |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | On every field change (draft save); on screen re-entry (draft restore); on Continue/Save while offline (queuing) |
| FEAT-01.SPEC-004 | Member List & Add Member | On screen re-entry, to reflect any queued member additions still pending sync |
| FEAT-01.SPEC-005 | Member Profile Detail | On every field change; on screen re-entry; on Save while offline |
| FEAT-01.SPEC-006 | Dietary Rules Editor | On every field change within the add/edit rule panel; on screen re-entry; on rule save while offline |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | On every field change; on screen re-entry; on Continue/Save while offline |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| draft_fields | No validation beyond data type -- a draft may hold partial or invalid values, since validation applies only at actual submission, not at draft-save time | Always | -- | -- | No |
| queued_operation | Must resolve to submitted or discarded -- a queued operation cannot remain indefinitely undecided once connectivity returns | When connectivity is restored | On reconnection | N/A -- resolution is automatic (see Business Rules), not a user-facing error | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Draft freshness on multi-device conflict | draft_fields, last_updated | If a draft exists locally on Device A and the same screen/target has also been saved (confirmed) from Device B since the local draft's last_updated, the confirmed save from Device B wins when Device A's screen reopens -- the stale local draft is discarded in favor of the current confirmed record, and Device A's user starts fresh from the confirmed state, not from their stale draft | N/A -- resolved silently, since the confirmed record is public state and always takes precedence over an unconfirmed local draft |

## Authorization Rules

{Draft persistence is a local, per-device, per-user mechanism -- there is no cross-role authorization surface here: a draft is always private to the device and account that created it, and the actions it governs (saving, resuming, queuing) inherit the authorization already defined by the screen the draft belongs to.}

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create/update a local draft | Maya (Organiser) | On any screen she is authorized to edit per FEAT-01.SPEC-016 | -- |
| Resume a local draft | Maya (Organiser) | Only on the same device/session where the draft was created; a draft never syncs to a different device before it is submitted | Opening the same setup screen on a different device shows the screen in its confirmed (last-saved) state, not the unsynced draft -- there is no error, since this is expected behavior, not a denial |
| Queue an offline save | Maya (Organiser) | Only for screens/actions she is authorized to perform per FEAT-01.SPEC-016 | An action Maya is not authorized to perform (e.g., Sam attempting an organiser-only edit) is blocked by FEAT-01.SPEC-016 before any draft or queue state is created |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| draft_screen | Set to the screen currently being edited | On first field change on any governed screen | No -- set automatically |
| last_updated | Current date and time | On every draft field change | No -- always current |
| queued_operation | False, unless a save is attempted while offline | On save attempt | No -- set automatically based on connectivity at save time |

## Business Rules

- A draft is created locally the moment the organiser begins typing or selecting on any governed screen -- it does not wait for a deliberate "save draft" action.
- Reopening a governed screen (via back navigation, closing and reopening the app, or after a session expiry per each screen's own Access and Visibility row) restores the most recent local draft for that screen/target, if one exists and no newer confirmed save has superseded it.
- A save attempted while offline is queued locally rather than failing outright; it submits automatically the moment connectivity returns, with no further action required from the organiser.
- A save that fails for a reason other than connectivity (e.g., a validation error surfaced by the server) is never silently discarded -- the entered data remains on screen with a retry option, per this feature's States field commitment ("a failed save keeps the entered data on screen and offers a retry, never silently discards input").
- Once a queued offline save submits successfully, the local draft for that screen/target is cleared, since the data is now part of the confirmed record.

## Edge Cases

- **Organiser closes the app mid-entry on FEAT-01.SPEC-005 with no connectivity at all** -- The draft is held locally; reopening the app (online or still offline) restores the exact field values as they were left.
- **Organiser is offline, queues a save on FEAT-01.SPEC-008, then edits the same fields again before connectivity returns** -- The queued save is updated in place to reflect the latest edit; only one submission occurs once connectivity returns, not one per edit.
- **Organiser signs in on a second device while a draft exists, unsynced, on the first device** -- The second device shows the last confirmed state only; the first device's draft remains local to it and is not visible elsewhere until it is submitted.
- **Connectivity returns and submits a queued save, but the underlying record was deleted or changed in a way that makes the queued save invalid** (e.g., the member being edited was removed via FEAT-18 while the device was offline) -- The queued submission fails with a clear message on next screen open: "This member no longer exists. Your changes could not be saved." rather than silently applying to a stale target.
- **Two queued saves for the same screen/target exist because the organiser used the app offline on two devices** -- On reconnection, each device submits independently; the dependency map's per-entity Contention resolution (last-write-wins for Household and Member Profile fields) determines the final state, consistent with those entities' own Contention notes rather than any special rule introduced here.
- **Organiser explicitly cancels an in-progress entry (e.g., taps "Cancel" with confirmation on FEAT-01.SPEC-005)** -- The local draft for that screen/target is discarded immediately; reopening the screen afterward starts fresh from the last confirmed state, not from the discarded draft.

## Acceptance Criteria

**FEAT-01.SPEC-013-AC-01:** Given Maya is entering a new member's name on FEAT-01.SPEC-005 and closes the app, when she reopens it and returns to that screen, then the entered name is still there exactly as she left it.

**FEAT-01.SPEC-013-AC-02:** Given Maya loses connectivity while filling in the weekly budget on FEAT-01.SPEC-008, when she taps "Continue", then the entry is queued locally and the banner confirms it will save once she reconnects.

**FEAT-01.SPEC-013-AC-03:** Given Maya's queued budget save is pending and connectivity returns, when the app detects the reconnection, then the save submits automatically with no action required from Maya.

**FEAT-01.SPEC-013-AC-04:** Given Maya's save fails due to a validation error rather than connectivity, when the failure occurs, then her entered data remains on screen with a retry option, never silently discarded.

**FEAT-01.SPEC-013-AC-05:** Given Maya has a local, unsynced draft on Device A, when she signs in on Device B and opens the same screen, then Device B shows only the last confirmed state, not Device A's draft.

**FEAT-01.SPEC-013-AC-06:** Given Maya is offline and queues a save, then edits the same fields again before reconnecting, when connectivity returns, then only one submission occurs, reflecting her latest edit.

**FEAT-01.SPEC-013-AC-07:** Given a queued save's target member was removed (FEAT-18) while Maya was offline, when connectivity returns and the queued save attempts to submit, then she sees "This member no longer exists. Your changes could not be saved." rather than the save silently applying incorrectly.

**FEAT-01.SPEC-013-AC-08:** Given Maya taps "Cancel" with confirmation on an in-progress member edit, when she confirms discarding, then the local draft is cleared and reopening the screen shows the last confirmed state.

**FEAT-01.SPEC-013-AC-09:** Given Maya has queued the same field's save from two different offline devices, when both reconnect, then the field resolves to the most recently saved value, consistent with the Household/Member Profile entities' own last-write-wins Contention rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Household & Member Field Validation Rules

## Overview

**Name:** Household & Member Field Validation Rules
**ID:** FEAT-01.SPEC-014
**Type:** Logic/Rule
**Purpose:** Governs household name, member cap, budget amount, and kid-profile data-minimality validation across every setup screen, plus account-level sign-in field rules.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles
**Governed Entity:** Household and Member Profile

## Scope and Non-Goals

**In Scope:**
- Field-level validation for household_name, weekly_budget, and the account sign-in fields (email, password) captured during account creation
- The 12-member household cap
- Member Profile field validation: display_name, age_band, and the kid-profile data-minimality constraint
- Authorization rules for creating and editing Household and Member Profile records

**Non-Goals:**
- Dietary Rule validation (allergen selection, strength, removal confirmation gate) -- governed entirely by FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules)
- Locale fields (unit_system, currency, aisle_names) and plan_arrival_day_time -- these Household fields are validated and owned by FEAT-16 and FEAT-07 respectively, per the dependency map's note that FEAT-01 does not update them
- Weekly schedule field validation -- the weekly_schedule field carries no validation beyond data type (an empty schedule is a valid, deliberate statement), as established in FEAT-01.SPEC-008

## Governed Entity

**Entity:** Household and Member Profile
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| household_name | text | The household's display name |
| organiser | reference | The one Member Profile holding the organiser role |
| weekly_budget | number | Rough weekly food budget, a positive amount in the household's currency |
| weekly_schedule | derived | Which nights are short on time and their time limit |
| unit_system | enum | Governed by FEAT-16, not this spec |
| currency | enum | Governed by FEAT-16, not this spec |
| aisle_names | text | Governed by FEAT-16, not this spec |
| plan_arrival_day_time | derived | Governed by FEAT-07, not this spec |
| status | enum | Active or Closed/Deleted; set only by FEAT-18, not this spec |
| display_name | text | First name or nickname (Member Profile) |
| member_type | enum | Organiser, Other Adult Member, young kid profile, or older kid limited login |
| sign_in | credential | Email and protected sign-in for adults only |
| age_band | enum | Kid profiles only |
| parental_consent_confirmation | boolean | Kid profiles only; required to create a kid profile |
| notification_preferences | derived | Per member: plan-ready on/off, nightly nudge on/off (adults only) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-001 | Account Sign-Up & Sign-In | On field blur and form submit for email and password; authorization is implicit (pre-authentication screen) |
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | On field blur and form submit for household_name; authorization on screen entry (organiser only) |
| FEAT-01.SPEC-004 | Member List & Add Member | On "Add" action attempt, for the 12-member cap; authorization on screen entry (Add/Invite hidden from Sam) |
| FEAT-01.SPEC-005 | Member Profile Detail | On field blur and form submit for display_name, age_band; authorization on screen entry and save |
| FEAT-01.SPEC-007 | Parental Consent Confirmation | On "Continue" tap, for parental_consent_confirmation |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | On field blur and form submit for weekly_budget |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Account email (FEAT-01.SPEC-001) | Required, valid email format | Always | On blur and on submit | "Enter a valid email address" | Yes |
| Account password (FEAT-01.SPEC-001) | Required, must meet a minimum protection strength (not trivially guessable) | On account creation only | On blur and on submit | "Choose a password that is harder to guess" | Yes |
| household_name | Required, 1-60 characters | Always | On blur and on submit | "Household name must be between 1 and 60 characters" | Yes |
| weekly_budget | If provided, must be a positive number greater than zero | Optional during partial setup; the rule applies only when a value is entered | On blur and on submit | "Enter an amount greater than zero" | Yes |
| display_name (Member Profile) | Required, 1-60 characters | Always | On blur and on submit | "Enter a name (up to 60 characters)" | Yes |
| member_type | Required, set once at creation and immutable afterward | Always | On creation only | N/A -- no editable field, no error state reachable after creation | Yes (at creation) |
| age_band | Required for kid profiles only | member_type is a kid profile | On blur and on submit | "Select an age band" | Yes |
| age_band | No validation beyond data type | member_type is an adult (Organiser or Other Adult Member) | -- | -- | -- |
| parental_consent_confirmation | Must be explicitly confirmed (checkbox checked) before a kid profile can be created | member_type is a kid profile | On the "Continue" action in FEAT-01.SPEC-007 | N/A -- the disabled "Continue" button prevents an invalid submission rather than surfacing an error | Yes |
| notification_preferences | No validation beyond data type -- any combination of on/off is valid | Always | -- | -- | No |
| organiser | Exactly one Active Member Profile must hold this role at all times | Always | Enforced structurally: creation always assigns it to the account holder; changing it is owned entirely by FEAT-09 | N/A -- this spec never presents an organiser-selection control; XBR-15 governs the hand-over path | Yes (structurally) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| 12-member household cap | Member Profile (count), Household | The total count of Active Member Profiles for a household cannot exceed 12 | "Your household already has 12 members, the most Plateful supports right now." |
| Kid-profile data minimality | member_type, display_name, age_band, parental_consent_confirmation | When member_type is a kid profile, no field beyond display_name, age_band, dietary rules, and parental_consent_confirmation may be captured or stored -- no surname, birth date, photo, or contact detail field exists for this member_type at all | N/A -- enforced by omission: no such field is ever presented or accepted for a kid profile, so no error state is reachable |
| Kid profile requires prior consent | member_type, parental_consent_confirmation | A Member Profile cannot be created with member_type set to a kid profile unless parental_consent_confirmation is already true, set by FEAT-01.SPEC-007 before this screen is reached | "A kid profile needs parent or guardian confirmation before it can be saved" -- shown only in the defensive case of a kid-profile creation attempted without having passed through FEAT-01.SPEC-007 |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create account | Maya, Sam (any adult, pre-authentication) | Always | -- |
| Create household | Maya (Organiser) | Only once per account (scope-boundaries.md SC-03); only for an account with no existing household | An account that already has a household is routed directly to FEAT-01.SPEC-010 rather than shown this screen at all |
| Edit household name | Maya (Organiser) | Always | Edit entry point not shown to Sam; a direct attempt shows "Only the organiser can change this" (FEAT-01.SPEC-010) |
| View household name | Maya (Organiser), Sam (Other Adult Member) | Always | -- |
| View household name | Riley (Operator, support) | Only through the separate read-only support view (FEAT-22), never directly | Riley never reaches this screen |
| View household name | Jordan (young kid, no login), Jordan (older kid, Later) | Never | N/A -- no login exists (young kid) or household setup is outside the older-kid login's entitlements |
| Add member (adult or kid) | Maya (Organiser) | Household has fewer than 12 Active members | Add actions remain visible but the create screen shows the 12-member-cap message when the cap is reached; Sam never sees Add controls at all |
| View member list | Maya (Organiser), Sam (Other Adult Member) | Always | -- |
| Edit a member's own display name/age band/type details | Maya (Organiser) | Any member | -- |
| Edit a member's own display name/age band/type details | Sam (Other Adult Member) | Never | Edit controls not shown; a direct attempt shows "Only the organiser can change member details" |
| Edit own notification preferences | Maya, Sam (each, their own) | Always, own record only | Attempting to edit another member's preference is not possible -- the control is not exposed for any profile but the signed-in adult's own |
| Set weekly_budget | Maya (Organiser) | Always | Screen not shown to Sam |
| View weekly_budget | Maya (Organiser), Sam (Other Adult Member) | Always | -- |
| Confirm parental consent | Maya (Organiser) | Always, and required before any kid profile creation | This confirmation cannot be granted by any other role; Sam has no entry point to FEAT-01.SPEC-007 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Household.organiser | The account holder creating the household | On household creation | No -- organiser status changes only through FEAT-09's hand-over |
| Household.status | Active | On creation | No |
| Member Profile.status | Active | On creation | No |
| Member Profile.member_type | Organiser, for the household's first (creating) member | On household creation | No |
| Household.weekly_budget | Unset (no value) until the organiser enters one | On creation, and until FEAT-01.SPEC-008 is completed | Yes -- the organiser sets or changes it at will |

## Business Rules

- The 12-member cap exists "comfortably above the brief's 2-6 people" (this feature's Validation & Limits), so it is intentionally generous rather than a tight operational constraint.
- Kid-profile data minimality is a compliance requirement, not a product preference: it enforces the children's-privacy-class protection described in assumptions-constraints.md ASMP-26/ASMP-27, and no downstream feature may add a field to the kid-profile data set that this spec does not already list.
- An allergy or religious rule, once entered on any member, is governed by FEAT-01.SPEC-015's removal confirmation gate, not by this spec -- this spec covers only the fields listed above.
- Field validation always runs before any cross-feature side effect (e.g., FEAT-01.SPEC-011's Subscription provisioning) -- a Household record is never created with an invalid household_name.

## Edge Cases

- **Household name entered as exactly 60 characters** -- Passes validation; 61 characters shows the length error.
- **Household name entered as only whitespace** -- Treated as empty; the "1-60 characters" required error applies, since whitespace-only input carries no meaningful household name.
- **Weekly budget entered as a very large number (e.g., far beyond any realistic grocery spend)** -- Passes validation; this spec sets no upper bound, since a rough guide has no meaningful ceiling worth blocking on.
- **Household at exactly 12 members, organiser attempts to add a 13th** -- Blocked with the exact cap message; the household's existing 12 members are unaffected.
- **A member profile is removed (FEAT-18), bringing the count below 12, then a new member is added** -- The 12-member cap is evaluated fresh at each add attempt against the current Active count, not any historical high-water mark.
- **Organiser attempts to set an age band on an adult profile via a manipulated request (defensive case)** -- age_band is not a field the interface ever presents for adult member_types; if such a value were somehow submitted, it is ignored rather than stored, since the field has no meaning outside kid profiles.
- **Kid-profile creation attempted with parental_consent_confirmation not yet true (defensive case, e.g., a stale or bypassed flow)** -- Blocked with "A kid profile needs parent or guardian confirmation before it can be saved"; the profile is not created.

## Acceptance Criteria

**FEAT-01.SPEC-014-AC-01:** Given Maya enters "T" as her household name (1 character), when she blurs the field, then no error appears, since 1 character satisfies the minimum.

**FEAT-01.SPEC-014-AC-02:** Given Maya enters a household name of 61 characters, when she blurs the field, then the error "Household name must be between 1 and 60 characters" appears.

**FEAT-01.SPEC-014-AC-03:** Given Maya enters a household name of only spaces, when she blurs the field, then the same length/required error appears, since whitespace-only input is treated as empty.

**FEAT-01.SPEC-014-AC-04:** Given Maya enters "0" for weekly_budget, when she blurs the field, then the error "Enter an amount greater than zero" appears.

**FEAT-01.SPEC-014-AC-05:** Given Maya leaves weekly_budget empty, when she proceeds, then no error appears, since the field is optional during partial setup.

**FEAT-01.SPEC-014-AC-06:** Given Maya's household has 12 Active members, when she attempts to add a 13th, then the error "Your household already has 12 members, the most Plateful supports right now." appears and no new member is created.

**FEAT-01.SPEC-014-AC-07:** Given a member is removed via FEAT-18, bringing the household to 11 members, when Maya adds a new member, then it succeeds, since the cap is evaluated against the current count.

**FEAT-01.SPEC-014-AC-08:** Given Maya is creating a kid profile, when she looks for a surname, birth date, photo, or contact field, then none exists on the screen at all.

**FEAT-01.SPEC-014-AC-09:** Given Maya attempts to save a kid profile without having completed FEAT-01.SPEC-007's confirmation (defensive case), then the save is blocked with "A kid profile needs parent or guardian confirmation before it can be saved."

**FEAT-01.SPEC-014-AC-10:** Given Sam (Other Adult Member) views a member's profile, when he looks for edit controls on display_name or age_band, then none are shown, and a direct edit attempt shows "Only the organiser can change member details."

**FEAT-01.SPEC-014-AC-11:** Given Maya (Organiser) edits any member's display name, when she saves, then the change is accepted regardless of which member it is.

**FEAT-01.SPEC-014-AC-12:** Given Sam edits his own notification preference toggle, when he saves, then only his own preference changes and Maya's is unaffected.

**FEAT-01.SPEC-014-AC-13:** Given a visitor enters an invalid email format when creating an account, when they blur the field, then the error "Enter a valid email address" appears.

**FEAT-01.SPEC-014-AC-14:** Given Maya attempts to create a second household on an account that already has one, then she is routed directly to FEAT-01.SPEC-010 and this creation screen is never shown.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 11 | 11 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Dietary Rule Classification & Allergen Matching Rules

## Overview

**Name:** Dietary Rule Classification & Allergen Matching Rules
**ID:** FEAT-01.SPEC-015
**Type:** Logic/Rule
**Purpose:** Governs allergen selection, hard-vs-soft strength classification, per-person vegetarian logic, and the explicit-confirmation gate before an allergy or religious rule is removed.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles
**Governed Entity:** Dietary Rule

## Scope and Non-Goals

**In Scope:**
- Allergen selection from the standard allergen list, plus the optional named-extra-ingredient field
- Strength classification: which rule_kind values are hard vs. soft
- Per-person vegetarian setting logic, including the shared-meal vegetarian-variant implication
- The explicit confirmation gate that must complete before an allergy or religious rule is removed
- Authorization for who can create, edit, and remove Dietary Rule entries

**Non-Goals:**
- Checking a Dietary Rule against any recipe's ingredients -- owned entirely by the Dietary Rules & Allergy Safety Engine (FEAT-02), which reads this data but performs no classification of its own
- The screen mechanics of entering or displaying rules -- owned by FEAT-01.SPEC-006 (Dietary Rules Editor), which enforces the rules defined here
- Medical or diet advice derived from a rule -- excluded per scope-boundaries.md SC-06; this spec classifies rules for matching purposes only, never as nutritional guidance

## Governed Entity

**Entity:** Dietary Rule
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| member | reference | The Member Profile the rule belongs to (required) |
| rule_kind | enum | Allergy, religious rule (e.g., halal), per-person vegetarian setting, or dislike |
| strength | enum | Hard (allergy, religious rule) or soft (dislike); vegetarian applies per person |
| allergen | reference/text | From a standard allergen list, optionally a named extra ingredient (required for allergies) |
| origin | enum | Entered by the organiser or learned from ratings (FEAT-12) |
| change_history | derived | Who changed the rule and when, visible to the organiser |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-006 | Dietary Rules Editor | On save of each rule entry (create/edit); on "Remove" tap for every rule_kind |
| FEAT-01.SPEC-012 | Mid-Week Hard-Rule Change Trigger | Reads this spec's hard/tightened classification to decide whether to fire |
| FEAT-02 | Dietary Rules & Allergy Safety Engine (cross-feature) | Reads strength and allergen classification when checking recipes; does not enforce this spec's field rules itself |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| member | Required, must reference an existing Active Member Profile in the household | Always | On save | N/A -- the field is set by the screen context (which member's editor is open), never entered directly; no error state is user-reachable | Yes |
| rule_kind | Required, one of: allergy, religious rule, vegetarian setting, dislike | Always | On save | "Choose what kind of rule this is" | Yes |
| allergen | Required, selected from the standard allergen list | rule_kind is allergy | On save | "Select an allergen from the list" | Yes |
| allergen (named extra ingredient) | Optional free-text, up to 80 characters | rule_kind is allergy | On save | "Ingredient name must be 80 characters or fewer" | Yes |
| allergen | No validation beyond data type -- not applicable | rule_kind is religious rule, vegetarian setting, or dislike | -- | -- | -- |
| Religious rule label | Required free-text, up to 60 characters | rule_kind is religious rule | On save | "Enter the religious rule (up to 60 characters)" | Yes |
| Dislike label | Required free-text, up to 60 characters | rule_kind is dislike | On save | "Enter the dislike (up to 60 characters)" | Yes |
| strength | Not user-entered -- derived automatically from rule_kind (see Defaults and Derivations) | Always | On save | N/A -- no error state, since the field is never directly editable | Yes (derived) |
| origin | Not user-entered on this screen -- set automatically to "entered by the organiser" for every rule created via FEAT-01.SPEC-006 | Always | On save | N/A -- FEAT-12 sets "learned from ratings" independently for its own creations | Yes (derived) |
| change_history | No validation beyond data type -- append-only, system-maintained | Always | On every create/edit/remove | N/A | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Strength follows rule_kind | rule_kind, strength | Allergy and religious rule always classify as hard; dislike always classifies as soft; vegetarian setting applies per person and is treated as hard for that person's own meal selection (a vegetarian member is never served a non-vegetarian dish), while never blocking the household's shared meal from existing, since a vegetarian-variant option can satisfy it | N/A -- strength is derived, not entered, so no invalid combination is directly reachable |
| Allergen required only for allergies | rule_kind, allergen | If rule_kind is allergy, allergen is required; for any other rule_kind, the allergen field is not shown at all | "Select an allergen from the list" (allergy only) |
| Duplicate allergen for the same member | member, rule_kind, allergen | Attempting to add a second allergy entry for an allergen the member already has routes to editing the existing entry rather than creating a duplicate | N/A -- handled by routing to edit, not by an error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create dietary rule | Maya (Organiser) | Any member of the household | -- |
| Create dietary rule (learned dislike) | System, via FEAT-12 | Origin set to "learned from ratings"; never created directly by Sam or any adult through this screen | N/A -- not a user-facing action on this spec's enforcing screen |
| View dietary rules | Maya (Organiser), Sam (Other Adult Member) | Always, for every member | -- |
| View dietary rules (allergy detail only) | Riley (Operator, support) | Only inside a specific open safety report (XBR-14), never the full list | Riley never reaches FEAT-01.SPEC-006 directly; broader dietary-rule visibility is never granted to the operator role |
| Edit dietary rule | Maya (Organiser) | Any member's rule | -- |
| Edit dietary rule | Sam (Other Adult Member) | Never | Rules render read-only; a direct interaction attempt shows "Only the organiser can change dietary rules" |
| Remove dislike | Maya (Organiser) | Any member's dislike, immediately, no confirmation gate | -- |
| Remove allergy or religious rule | Maya (Organiser) | Only after completing the explicit confirmation gate (see Business Rules) | Attempting removal without confirming shows the confirmation modal; the rule is not removed until the modal's explicit affirmative action is taken |
| Remove allergy or religious rule while offline | Maya (Organiser) | Never -- the confirmation gate requires a live check against the current record | "Removing an allergy needs a connection -- try again once you're back online." |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| strength | Derived from rule_kind: allergy -> hard; religious rule -> hard; vegetarian setting -> hard (for that member's own selection); dislike -> soft | On every create/edit | No -- never directly editable |
| origin | "Entered by the organiser" | On creation via FEAT-01.SPEC-006 | No |
| change_history | A new entry is appended: who made the change, what changed, and when | On every create, edit, and remove | No -- always system-maintained, never editable or deletable by any role |

## Business Rules

- An allergy or religious rule can never be removed without the organiser completing an explicit confirmation step first: the confirmation modal states exactly what will happen ("Removing this will let recipes containing {allergen/rule} be suggested again for {member}") and requires an explicit affirmative tap, with no default-confirmed state, per this feature's Shared UI Patterns.
- A dislike can be removed directly with no confirmation step, since it is soft and removing it carries no safety consequence.
- A learned soft dislike (origin: learned from ratings, created by FEAT-12) never overwrites or weakens an explicit organiser-entered rule for the same ingredient; an explicit organiser edit always takes precedence over a learned entry for the same ingredient, per the dependency map's Contention note for Dietary Rule.
- A new or tightened hard rule (allergy or religious rule) triggers FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) -- this spec defines what qualifies as "tightened" (a widened allergen scope or an added named ingredient), while FEAT-01.SPEC-012 owns the resulting re-check.
- A rule's change_history entry survives the rule's own removal indefinitely, per this feature's Data Notes: "Each dietary rule's change history is kept for trust and any safety investigation," even though the rule itself is gone with no restore path.
- Vegetarian setting is per-person, not household-wide: a shared meal in a mixed-diet household can carry a vegetarian-variant option to satisfy a vegetarian member without requiring every other member to eat vegetarian.

## Edge Cases

- **Organiser selects an allergen already on the member's list and attempts to add it again** -- The add-rule panel routes to editing the existing entry instead of creating a duplicate Dietary Rule for the same allergen.
- **Organiser names an extra ingredient at exactly 80 characters** -- Passes validation; 81 characters shows the length error.
- **Organiser attempts to remove an allergy while offline** -- Blocked with the connectivity message; the confirmation modal never opens, since the gate requires a live check.
- **A learned soft dislike (FEAT-12) targets the same ingredient as an organiser-entered explicit rule** -- The explicit rule's strength and presence are unaffected; the learned entry exists as a separate soft record that influences selection weighting only, never overriding the explicit rule.
- **Organiser attempts to remove an allergy, confirms the modal, but connectivity is lost between confirmation and completion** -- The removal does not complete; the rule remains present, and the organiser sees a retry option, consistent with a failed-save-preserves-state posture.
- **Two hard rules for the same member are added in immediate succession (e.g., an allergy and a religious rule)** -- Each triggers its own independent evaluation by FEAT-01.SPEC-012; both re-checks may run, and a meal failing either is removed once, not twice, since FEAT-02 evaluates the full current rule set on each re-check rather than one changed rule in isolation.
- **Vegetarian setting toggled on for a member in a household with no other vegetarian members** -- The rule is recorded exactly the same as any other member's vegetarian setting; no household-level change is required, since the setting is inherently per-person.

## Acceptance Criteria

**FEAT-01.SPEC-015-AC-01:** Given Maya selects "Peanuts" from the standard allergen list for Jordan and saves, then a Dietary Rule is created with rule_kind=allergy, allergen=Peanuts, strength=hard.

**FEAT-01.SPEC-015-AC-02:** Given Maya adds a dislike for a member and saves, then the rule is created with strength=soft.

**FEAT-01.SPEC-015-AC-03:** Given Maya toggles the vegetarian setting on for a member and saves, then the rule is recorded with strength=hard for that member's own meal selection, without requiring the household's shared meal to be vegetarian.

**FEAT-01.SPEC-015-AC-04:** Given Maya attempts to save an allergy with no allergen selected, when she taps Save, then the error "Select an allergen from the list" appears and no rule is created.

**FEAT-01.SPEC-015-AC-05:** Given Maya taps "Remove" on a dislike, then it is removed immediately with no confirmation step.

**FEAT-01.SPEC-015-AC-06:** Given Maya taps "Remove" on an existing allergy, then a confirmation modal appears stating what will happen, and the allergy is not removed until she taps the explicit affirmative action.

**FEAT-01.SPEC-015-AC-07:** Given Maya is offline and taps "Remove" on a religious rule, then she sees "Removing an allergy needs a connection -- try again once you're back online." and the confirmation modal does not open.

**FEAT-01.SPEC-015-AC-08:** Given a learned soft dislike from FEAT-12 targets the same ingredient as Maya's existing explicit allergy, when both exist, then the explicit allergy's strength and presence are unaffected by the learned entry.

**FEAT-01.SPEC-015-AC-09:** Given Maya attempts to add a second allergy entry for peanuts when one already exists for that member, then she is routed to edit the existing entry rather than a duplicate being created.

**FEAT-01.SPEC-015-AC-10:** Given Maya names a specific extra ingredient at exactly 80 characters, when she saves, then the rule is created successfully.

**FEAT-01.SPEC-015-AC-11:** Given Maya names a specific extra ingredient at 81 characters, when she saves, then the error "Ingredient name must be 80 characters or fewer" appears.

**FEAT-01.SPEC-015-AC-12:** Given Sam views any member's dietary rules, when he attempts to interact with a rule, then no edit or remove control is reachable and he sees "Only the organiser can change dietary rules" if he tries a direct action.

**FEAT-01.SPEC-015-AC-13:** Given Maya confirms the removal modal for an allergy but connectivity is lost before the removal completes, then the allergy remains present and Maya is offered a retry once connectivity returns.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Household Setup Authorization Rules

## Overview

**Name:** Household Setup Authorization Rules
**ID:** FEAT-01.SPEC-016
**Type:** Logic/Rule
**Purpose:** Governs who can view or change each part of household setup, including kid-profile data restrictions and the unauthorized-visitor experience, as the single authoritative home for role-gated behavior across every setup screen.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles
**Governed Entity:** Household and Member Profile (the access dimension -- which role may view or act on each, across every setup screen FEAT-01.SPEC-001 through FEAT-01.SPEC-010)

## Scope and Non-Goals

**In Scope:**
- The complete role-action matrix for every action this feature defines on Household and Member Profile data
- The exact unauthorized experience for every role denied an action
- What an unauthenticated visitor and an expired session see across this feature's screens

**Non-Goals:**
- Dietary Rule-specific authorization (who may edit/remove a rule) -- governed by FEAT-01.SPEC-015, which is more specific to that entity's own removal-confirmation gate
- Authorization for features outside this one (e.g., who may approve a Weekly Plan) -- each feature owning its own entities defines its own authorization; this spec covers only Household and Member Profile actions within FEAT-01
- Operator (Riley) authentication or the mechanics of the read-only support view itself -- owned by FEAT-22 (Operator Read-Only Support Access); this spec only states that Riley's access to household setup facts is View-only and delivered exclusively through that separate view, per XBR-14

## Governed Entity

**Entity:** Household and Member Profile (access dimension)
**Source:** Feature Dependency Map; Access Matrix in user-persona.md

| Field | Data Type | Description |
|-------|-----------|-------------|
| (No new fields -- this spec governs actions on the existing Household and Member Profile fields already defined in FEAT-01.SPEC-014) | -- | -- |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-001 | Account Sign-Up & Sign-In | On screen entry (pre-authentication) and on sign-in routing |
| FEAT-01.SPEC-002 | Password Recovery | On screen entry (pre-authentication) |
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | On screen entry and on save |
| FEAT-01.SPEC-004 | Member List & Add Member | On screen entry (Add/Invite visibility) and on save |
| FEAT-01.SPEC-005 | Member Profile Detail | On screen entry (edit vs. read-only) and on save |
| FEAT-01.SPEC-006 | Dietary Rules Editor | On screen entry (edit vs. read-only) |
| FEAT-01.SPEC-007 | Parental Consent Confirmation | On screen entry (organiser only) |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | On screen entry (organiser only) and on save |
| FEAT-01.SPEC-009 | Setup Complete & Next Steps | On screen entry (organiser only) |
| FEAT-01.SPEC-010 | Household Settings Hub | On screen entry (row visibility per role) |

## Field Validation Rules

N/A -- this spec governs authorization (who may act), not field-level data validation, which is FEAT-01.SPEC-014's and FEAT-01.SPEC-015's scope. No field in the governed entity carries validation rules distinct from those already defined there.

## Cross-Field Rules

N/A -- no cross-field data rule applies at the authorization layer; every rule here is a role-action-condition triple, captured in full in the Authorization Rules table below.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Sign up / sign in | Maya, Sam (any adult) | Always, pre-authentication | -- |
| Create household | Maya (Organiser) | Account has no existing household | An account that already has a household is routed directly to FEAT-01.SPEC-010, never shown FEAT-01.SPEC-003 |
| View/edit household name, budget, schedule | Maya (Organiser) | Full, always | -- |
| View household name, budget, schedule | Sam (Other Adult Member) | View only, always | Edit entry points not shown; a direct attempt shows "Only the organiser can change this" |
| View household name, budget, schedule | Jordan (young kid profile, no login), Jordan (older kid, limited login) | Never | N/A -- no login exists (young kid); household setup is outside the older-kid login's entitlements |
| View household name, budget, schedule | Riley (Operator, support) | View only, exclusively through the separate read-only support view (FEAT-22), from v1 | Riley never reaches this feature's own screens directly |
| Add/invite members | Maya (Organiser) | Household has fewer than 12 Active members | Add/Invite controls not shown to Sam at all; a direct navigation attempt shows "Only the organiser can add members" |
| View member list | Maya (Organiser), Sam (Other Adult Member) | Always | -- |
| Edit a member's own display name/age band/type | Maya (Organiser) | Any member | -- |
| Edit a member's own display name/age band/type | Sam (Other Adult Member) | Never | Edit controls not shown; a direct attempt shows "Only the organiser can change member details" |
| View a kid profile's details and dietary rules | Sam (Other Adult Member) | View only, always | -- |
| Edit a kid profile's details or dietary rules | Sam (Other Adult Member) | Never | Fields render read-only with the note "Only the organiser can change member details" (FEAT-01.SPEC-005) or "Only the organiser can change dietary rules" (FEAT-01.SPEC-006) |
| Edit own notification preferences | Maya, Sam (each, own record only) | Always | Editing another member's preference is never exposed as a control |
| Confirm parental consent for a kid profile | Maya (Organiser) | Always, required before kid-profile creation | No other role has an entry point to this action |
| View kid profile data | Riley (Operator, support) | Never, except the allergy details inside a specific open safety report (XBR-14) | Riley never sees a kid profile's full detail; broader visibility is structurally absent from the support view |

## Defaults and Derivations

N/A -- this spec assigns no default or derived field values of its own; all defaults for Household and Member Profile fields are defined in FEAT-01.SPEC-014.

## Business Rules

- Every screen in this feature (FEAT-01.SPEC-001 through FEAT-01.SPEC-010) references this spec for its role-gated behavior rather than restating access rules per screen, per this feature's Shared Validation section.
- An unauthorized visitor -- anyone not signed in as a member of the household -- sees only a sign-in screen (FEAT-01.SPEC-001), an invitation-acceptance screen for a link addressed to them (owned by FEAT-09), or the public welcome page of a household referral link (owned by FEAT-24) -- never household data, per user-persona.md's Access Matrix notes.
- A session that expires mid-setup returns the user to FEAT-01.SPEC-001 with the message "Your session has expired. Sign in to continue."; any unsaved draft is preserved per FEAT-01.SPEC-013 and restored to the same screen after re-authentication.
- Riley's (Operator) access to any fact this feature owns is always View-only and delivered exclusively through the separate read-only support view (FEAT-22, from v1) -- never through this feature's own screens, and every such visit is recorded where the organiser can see it (XBR-14).
- Roles named in this spec trace exactly to the Access Matrix in user-persona.md: Maya (Organiser), Sam (Other Adult Member), Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later), and Riley (Operator, support, from v1). No role beyond these five is ever introduced by this feature.

## Edge Cases

- **An unauthenticated visitor attempts to navigate directly to FEAT-01.SPEC-004 (Member List)** -- Redirected to FEAT-01.SPEC-001; no household data is ever rendered, even momentarily, during the redirect.
- **Sam's session expires while he is viewing a kid profile's read-only dietary rules** -- He is redirected to FEAT-01.SPEC-001 with the expired-session message; since he had no unsaved edits (his access is read-only there), nothing needs to be preserved.
- **Riley's operator session somehow reaches a FEAT-01 URL directly (defensive case)** -- No such entry point exists for the operator role in this feature; household data is never rendered to an operator session outside FEAT-22's own read-only support view.
- **The organiser role changes hands mid-session (FEAT-09 hand-over completes while the former organiser has a setup screen open)** -- The former organiser's next action against an organiser-only control (e.g., attempting to save a budget change) is rejected with "Only the organiser can change this," since authorization is evaluated at action time, not at screen-load time.
- **A kid profile somehow attempts a direct action (defensive case, e.g., a stale token)** -- No login exists for a young kid profile, so no valid session can be associated with one; no action succeeds.
- **Sam attempts to bypass the hidden Add/Invite controls via a direct navigation URL** -- The destination screen itself enforces the same authorization check on entry, showing "Only the organiser can add members" regardless of how the screen was reached.

## Acceptance Criteria

**FEAT-01.SPEC-016-AC-01:** Given an unauthenticated visitor attempts to open FEAT-01.SPEC-010 directly, when the request is made, then they are redirected to FEAT-01.SPEC-001 and see no household data.

**FEAT-01.SPEC-016-AC-02:** Given Maya (Organiser) is signed in, when she opens any FEAT-01 screen, then she has full view and edit access per the Authorization Rules table.

**FEAT-01.SPEC-016-AC-03:** Given Sam (Other Adult Member) is signed in, when he opens FEAT-01.SPEC-004, then he sees the member list but no Add or Invite controls.

**FEAT-01.SPEC-016-AC-04:** Given Sam attempts to navigate directly to a member edit screen, when the screen loads, then he sees "Only the organiser can change member details" and no edit control is reachable.

**FEAT-01.SPEC-016-AC-05:** Given Sam views a kid profile's dietary rules, when the screen loads, then every rule renders read-only.

**FEAT-01.SPEC-016-AC-06:** Given Maya's session expires while she is mid-edit on FEAT-01.SPEC-008, when she is redirected, then she sees "Your session has expired. Sign in to continue." and her unsaved entry is restored after she signs back in.

**FEAT-01.SPEC-016-AC-07:** Given Riley (Operator) is using the separate read-only support view (FEAT-22), when he views household facts, then he sees them as View-only and never through any FEAT-01 screen directly.

**FEAT-01.SPEC-016-AC-08:** Given a young kid profile has no login, when any request is made under that profile's identity, then no action succeeds, since no valid session can exist for it.

**FEAT-01.SPEC-016-AC-09:** Given the organiser role is handed over to Sam via FEAT-09 while Maya still has FEAT-01.SPEC-008 open, when Maya (now a non-organiser) attempts to save a budget change, then it is rejected with "Only the organiser can change this."

**FEAT-01.SPEC-016-AC-10:** Given an unauthorized visitor follows a household referral link (FEAT-24), when the welcome page loads, then they see only the inviter's first name and no other household data.

**FEAT-01.SPEC-016-AC-11:** Given Sam attempts to reach the add-member flow via a direct URL, then the destination screen itself blocks the action with "Only the organiser can add members," regardless of the navigation path used.

**FEAT-01.SPEC-016-AC-12:** Given Riley's session somehow constructs a direct request to a FEAT-01 screen, then no household data is returned, since the operator role has no entry point into this feature's own screens.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- covered by FEAT-01.SPEC-014) | 0 |
| Cross-Field Rules | 0 (N/A) | 0 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 0 (N/A) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Integration Spec: Transactional Email Integration (Account & Recovery)

## Overview

**Name:** Transactional Email Integration (Account & Recovery)
**ID:** FEAT-01.SPEC-017
**Type:** Integration
**Purpose:** Sends the account-creation confirmation email and the sign-in recovery email through the transactional email capability.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Sending the account-confirmation email when a new account is created
- Sending the reset-link email when a sign-in reset is requested
- User-facing behavior when the transactional email capability is slow, unavailable, or rejects a send
- Disclosure of what account data is shared with the capability to deliver these emails

**Non-Goals:**
- Choosing the email-delivery vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Any other transactional email this product sends (safety-report emails, plan-ready email fallback, billing confirmations, export/deletion emails) -- each is owned by the feature whose Communications require it (FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-14.SPEC-012, FEAT-18.SPEC-012 respectively, per the Feature Dependency Map's External Touchpoints table); this spec covers only the account-creation and recovery emails named in this feature's own Communications
- The screen mechanics of requesting account creation or a reset -- owned by FEAT-01.SPEC-001 and FEAT-01.SPEC-002; this spec defines only the capability behavior those screens trigger

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Requires transactional email for account sign-up and sign-in recovery" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this spec, FEAT-01.SPEC-017, covers the account & recovery portion)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A newly created account receives a confirmation that their account exists | Create an account and sign in | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| An adult who cannot sign in receives a reset link by email | Create an account and sign in (recovery) | FEAT-01.SPEC-002 (Password Recovery) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Account email address | Member Profile -- sign_in (email component) | Account is created | The capability needs a destination address to deliver the confirmation |
| Reset token and its expiry | Derived -- a single-use, time-limited token tied to the account | A sign-in reset is requested | The capability delivers a link the account holder uses to complete the reset |

No other Member Profile or Household field ever leaves the product through this integration. Dietary Rule data, household facts, and any other account holder's data are never included in these emails.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / bounced / failed) | The capability reports the send result | No entity field is updated by a successful delivery; a bounced or failed send updates an internal delivery-status flag on the pending email attempt (not a Household or Member Profile field), used only to decide whether to retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Confirmation email delivered | The capability confirms the account-confirmation email reached the recipient's inbox | None -- delivery confirmation is not surfaced as a user-visible change | None -- FEAT-01.SPEC-001 already proceeded to setup regardless of delivery, per this feature's non-blocking design | FEAT-01.SPEC-001 |
| Reset email delivered | The capability confirms the reset-link email reached the recipient's inbox | None | None -- FEAT-01.SPEC-002 already shows the neutral confirmation message regardless of delivery status | FEAT-01.SPEC-002 |
| Send failed | The capability reports it could not deliver either email (bounced, rejected, or a hard failure) | The pending email attempt's internal delivery-status flag is set to failed | None immediately -- see Degradation Behavior; the account-creation or reset-request screen flow is not blocked by this outcome | FEAT-01.SPEC-001, FEAT-01.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | No user-visible effect -- account creation completes and the organiser proceeds to FEAT-01.SPEC-003 regardless of how long the confirmation email takes to send | No user-visible effect -- account creation is not blocked by this capability being unavailable; the confirmation email is queued to send once the capability recovers | No user-visible effect on this screen -- a rejected send (e.g., an invalid-looking address) does not prevent account creation or setup from proceeding, since the email is a courtesy confirmation, not a required verification gate |
| FEAT-01.SPEC-002 (Password Recovery) | No user-visible effect on the request step -- the neutral confirmation message appears regardless of send speed, so timing differences never disclose whether an account exists | No user-visible effect on the request step -- the same neutral confirmation appears; the reset email is queued to send once the capability recovers. If the capability remains down long enough that no reset link ever arrives, the adult sees no error (consistent with never disclosing account existence) and can request again later | N/A -- a rejected send at the request step produces the same neutral confirmation as any other outcome, by design, so no rejection state is ever distinguishable to the user here |

## Consent and Disclosure

- **Account confirmation email disclosure** -- The account-creation confirmation on FEAT-01.SPEC-001 implies, as standard product behavior, that a confirmation email will be sent to the address just provided; no separate consent prompt interrupts account creation, since sending an account holder a message at their own just-provided address is expected behavior for creating an account, not a data-sharing decision requiring a choice.
- **Reset email disclosure** -- The confirmation message on FEAT-01.SPEC-002, "If an account exists for {entered email}, a reset link is on its way. Check your inbox.", is itself the disclosure that an email will be sent to that address if it matches an account.
- **What is never shared** -- Household facts, Dietary Rule data (including any child's allergy information), and any other member's data are never included in the account-confirmation or reset emails; only the account holder's own email address and a system-generated reset token leave the product through this integration.

## Edge Cases

- **Reset email event arrives for an account that was deleted between the request and the send** -- The event is discarded silently; no email is sent for a deleted account, and no user feedback fires, since the requester received the neutral confirmation regardless of outcome.
- **The same "send failed" event is delivered twice for one attempt** -- The second delivery changes nothing: the delivery-status flag is already failed, and no duplicate retry is triggered beyond the single retry policy already in effect.
- **Confirmation and failure events arrive out of order (failure reported, then a late "delivered" event for the same attempt)** -- The most recent event by its own timestamp governs the delivery-status flag; a late "delivered" event arriving after a "failed" event corrects the flag back to delivered, since it reflects a true, if delayed, outcome.
- **Capability goes down mid-send for a reset email** -- If the send was not confirmed initiated, the reset request is treated as not yet sent and is queued for retry once the capability recovers; the requester's neutral confirmation message is unaffected either way.
- **An adult requests a reset twice in quick succession before the first email's send confirms** -- Both are queued; per FEAT-01.SPEC-002's business rules, the newer link becomes the valid one, so only the most recent reset token matters even if both emails eventually deliver.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Triggered by (inbound) | Successful account creation triggers the confirmation email |
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Affects (outbound) | Degradation behavior surfaces here (as no user-visible effect, by design) |
| FEAT-01.SPEC-002 (Password Recovery) | Triggered by (inbound) | A reset request triggers the reset-link email |
| FEAT-01.SPEC-002 (Password Recovery) | Affects (outbound) | Degradation behavior and the neutral confirmation message surface here |

## Analytics and Success Signals

- **account_confirmation_email_sent** (delivery outcome: delivered / failed) -- N/A -- no Stage 2 metric measures confirmation-email delivery directly; retained as an operational signal for email-capability health.
- **reset_email_sent** (delivery outcome: delivered / failed) -- supports success-metrics.md: "First-Session Onboarding Completion" (a returning organiser blocked from signing in depends on this email arriving to resume setup or return to their household).

## Acceptance Criteria

**FEAT-01.SPEC-017-AC-01:** Given Maya creates a new account on FEAT-01.SPEC-001, when the account is created, then this integration sends an account-confirmation email to her registered address.

**FEAT-01.SPEC-017-AC-02:** Given Sam requests a sign-in reset on FEAT-01.SPEC-002 for his registered email, when the request is submitted, then this integration sends a reset-link email to that address.

**FEAT-01.SPEC-017-AC-03:** Given the transactional email capability is slow, when Maya creates an account, then account creation and her navigation to FEAT-01.SPEC-003 are unaffected by the delay.

**FEAT-01.SPEC-017-AC-04:** Given the transactional email capability is down, when Sam requests a reset, then he still sees the neutral confirmation message and the email is queued to send once the capability recovers.

**FEAT-01.SPEC-017-AC-05:** Given the transactional email capability rejects the send for an account-confirmation email, when this occurs, then Maya's account creation and setup flow proceed unaffected.

**FEAT-01.SPEC-017-AC-06:** Given a reset-email send event arrives for an account that was deleted since the request, when this integration processes it, then no email is sent and no user feedback fires.

**FEAT-01.SPEC-017-AC-07:** Given a "send failed" event for a reset email is delivered twice, when the second delivery arrives, then the delivery-status flag remains failed and no duplicate retry beyond the standard policy occurs.

**FEAT-01.SPEC-017-AC-08:** Given a "delivered" event arrives after an earlier "failed" event for the same reset email attempt, when it is processed, then the delivery-status flag corrects to delivered, since the later event by timestamp governs.

**FEAT-01.SPEC-017-AC-09:** Given an adult requests a reset twice in quick succession, when both emails are eventually sent, then only the most recently generated reset token is valid, per FEAT-01.SPEC-002's business rules.

**FEAT-01.SPEC-017-AC-10:** Given Maya has never seen a data-sharing disclosure interrupt account creation, when she reviews the confirmation message on FEAT-01.SPEC-002, then she recognizes it as the disclosure that a reset email will be sent if her entered address matches an account.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 6 (2 screens x 3 conditions) | 6 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |



# Notification Spec: Mid-Week Rule Change Notification

## Overview

**Name:** Mid-Week Rule Change Notification
**ID:** FEAT-01.SPEC-018
**Type:** Notification
**Purpose:** Tells the organiser which plan meal was removed after a mid-week hard dietary-rule change, so she is never left wondering why a night's dinner disappeared.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- The notification sent when FEAT-01.SPEC-012's re-check removes one or more meals from the current week's plan
- The single-removal and multiple-removal (batched) content variants
- Preference, retry, and expiry behavior for this notification

**Non-Goals:**
- Deciding which meals are removed -- owned by FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) and FEAT-02 (Dietary Rules & Allergy Safety Engine); this spec begins where that decision's outcome is handed to it
- Offering a replacement for the removed meal -- owned by One-Tap Meal Swap (FEAT-04), which the notification's call to action deep-links into
- Any other safety-related notification (e.g., a safety-concern report's acknowledgement, owned by FEAT-02.SPEC-010) -- this spec covers only the mid-week rule-change removal named in this feature's own Communications field

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a removal occurs | Maya's plan lives in-app; the removal is a change to something she can act on immediately (swap in a replacement), so it must appear where that action happens |
| Email | When the organiser's plan-ready email fallback preference indicates she is not reliably reached by device notification (mirroring FEAT-07's channel logic) | A safety-driven plan change is significant enough that it should not depend solely on Maya having the app open; the same fallback logic already established for plan-ready notifications (FEAT-07) applies here for consistency |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Meal(s) removed by mid-week re-check | FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) | Fires whenever the re-check removes one or more Planned Meals from the current week's plan | Household reference, the removed meal(s)' night and recipe name, the member and rule that triggered the change |

## Audience and Preferences

**Recipients:** Maya (Organiser) -- the sole recipient, per the Access Matrix in user-persona.md: this feature's Household Setup and Weekly Plan access is Full for the organiser and View for Sam, and the Communications field states specifically "the organiser is also told," not the whole household. Sam is not a recipient of this notification, since plan-change communications for other adults are scoped elsewhere (e.g., swap-suggestion outcomes in FEAT-04).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Plan-ready notifications (shared toggle also governing this notification's in-app/email channel choice) | On / Off | On | FEAT-01.SPEC-005 (Member Profile Detail), FEAT-07 |

**Quiet Hours:** N/A -- a safety-driven plan removal is delivered immediately regardless of quiet hours, since leaving the organiser unaware that a meal disappeared from her plan for longer than necessary works against the product's zero-incident safety commitment; this is a deliberate exception to the quiet-hours behavior that governs less time-sensitive notifications like FEAT-07's plan-ready message.

## Content Definition

**In-app (single meal removed):**
- **Title:** A meal was removed from your plan
- **Body:** {night}'s {recipe_name} was removed because of {member_name}'s updated {rule_label}.
- **CTA:** Choose a replacement -- deep-links to FEAT-04 (One-Tap Meal Swap) for the open {night} slot

**Email (single meal removed):**
- **Subject:** Your plan changed: {recipe_name} removed
- **Body:**
  Hi {organiser_first_name},

  We removed {recipe_name} from {night} because it's no longer safe for {member_name}'s updated {rule_label}.

  Open your plan to choose a safe replacement for that night.
- **CTA (button):** Choose a replacement -- deep-links to FEAT-04 (One-Tap Meal Swap) for the open {night} slot

**In-app (batched, 2+ meals removed):**
- **Title:** {count} meals were removed from your plan
- **Body:** {night_list} were removed because of {member_name}'s updated {rule_label}.
- **CTA:** Review your plan -- deep-links to FEAT-03 (AI Weekly Dinner Plan Generation) or FEAT-23 (Manual Weekly Planning), whichever produced the current plan, showing the affected week

**Email (batched, 2+ meals removed):**
- **Subject:** Your plan changed: {count} meals removed
- **Body:**
  Hi {organiser_first_name},

  We removed {count} meals from this week's plan because they're no longer safe for {member_name}'s updated {rule_label}: {night_list}.

  Open your plan to choose safe replacements.
- **CTA (button):** Review your plan -- deep-links to FEAT-03 or FEAT-23, showing the affected week

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {night} | Planned Meal -- night | Thursday | Never empty -- night is required on every Planned Meal |
| {recipe_name} | Recipe -- name | Spinach and Feta Pasta | Never empty -- name is required on every Recipe |
| {member_name} | Member Profile -- display_name | Jordan | Never empty -- display_name is required on every Member Profile |
| {rule_label} | Dietary Rule -- rule_kind and allergen (or religious-rule label) | peanut allergy | "dietary rule" (a generic label, used only if the specific rule's label cannot be resolved at send time) |
| {organiser_first_name} | Member Profile (organiser) -- display_name | Maya | "there" (a neutral greeting substitute) |
| {count} | Derived -- number of meals removed by this re-check | 3 | Never empty -- the batched variant only renders with 2 or more removed meals |
| {night_list} | Derived -- comma-separated list of removed nights | Wednesday and Thursday | Never empty -- the batched variant only renders with 2 or more removed meals |

## Delivery Rules

**Batching:** All meals removed by a single mid-week re-check run (FEAT-01.SPEC-012) are delivered as one notification, using the batched variant when 2 or more meals are removed by that run. Meals removed by a later, separate rule change (e.g., a second hard rule added the next day) generate their own, separate notification -- removals from different re-check runs are never merged into one message.
**Deduplication:** At most one notification per re-check run. A re-check that removes zero meals never generates a notification. If the same meal is somehow flagged by two rules evaluated in the same re-check run (e.g., it violates both a new allergy and a tightened religious rule), it is named once in the single resulting notification, not twice.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, mirroring FEAT-07's plan-ready email fallback pattern; after the final failure, the in-app notification stands as the delivery of record, and no alarming failure message is shown to the organiser. In-app delivery has no retry: it is delivered when the organiser next opens the product, and the underlying plan change is visible in-app regardless of whether the notification itself was seen.
**Expiry:** This notification does not expire in the sense of becoming irrelevant -- the plan change it describes is permanent (no restore path, per this feature's Entity-Lifecycle Coverage Matrix for Dietary Rule and Planned Meal), so the notification remains meaningful and deliverable whenever it is eventually seen. There is no cutoff after which it is withheld.

## Edge Cases

- **The removed meal's recipe is itself later removed from the library entirely (FEAT-10, unrelated event)** -- The notification's content is generated at the moment of removal and does not re-resolve {recipe_name} later, so an already-sent or already-queued notification is unaffected by the recipe's later removal from the library.
- **The organiser turns off plan-ready notifications between the re-check firing and delivery** -- Per XBR-13, each member controls their own notification preferences; if Maya turns this shared preference off before delivery, the pending notification is still delivered, since a safety-driven plan removal is exempt from the ordinary preference-timing rule that governs less time-sensitive notifications, given the zero-incident safety commitment this notification serves.
- **Quiet hours would otherwise apply** -- Not applicable, per the Quiet Hours section above: this notification is exempt and always delivers immediately.
- **The organiser's account is removed or the household is deleted (FEAT-18) between the re-check and delivery** -- The notification is cancelled silently on every channel; a household that no longer exists has no plan to review.
- **Two hard-rule changes trigger two separate re-check runs within moments of each other, each removing a different meal** -- Each run generates its own notification per the Deduplication rule; the organiser receives two separate notifications rather than one merged one, since they originate from two distinct re-check runs.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) | Triggered by (inbound) | A re-check that removes one or more meals fires this notification |
| FEAT-01.SPEC-005 (Member Profile Detail) | References (inbound) | Plan-ready preference toggle governs this notification's delivery channel |
| FEAT-07 (Weekly Plan Ready Notification) | References (inbound) | Shares the same preference and email-fallback channel logic |
| FEAT-04 (One-Tap Meal Swap) | Navigation (outbound) | Single-meal CTA deep-links here |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (outbound) | Batched-meal CTA deep-links here for a generated plan |
| FEAT-23 (Manual Weekly Planning) | Navigation (outbound) | Batched-meal CTA deep-links here for a manually built plan |

## Analytics and Success Signals

- **midweek_rule_change_notification_delivered** (channel: in_app / email; batched: yes / no; removed_meal_count) -- supports success-metrics.md: "Zero Allergy Incidents" (confirms the organiser is reliably told every time a safety-driven removal occurs, which is the transparency half of the zero-incident promise).
- **midweek_rule_change_notification_opened** (channel) -- N/A -- no distinct Stage 2 metric measures notification open rate for this specific message; retained to distinguish delivery from the organiser actually seeing it.
- **midweek_rule_change_cta_tapped** (destination: swap / plan_review) -- supports success-metrics.md: "Zero Allergy Incidents" (measures whether the organiser closes the loop by choosing a safe replacement after being told).

## Acceptance Criteria

**FEAT-01.SPEC-018-AC-01:** Given a mid-week re-check (FEAT-01.SPEC-012) removes one meal from Maya's plan, when the removal completes, then Maya receives an in-app notification titled "A meal was removed from your plan" naming the removed recipe, night, member, and rule.

**FEAT-01.SPEC-018-AC-02:** Given Maya's plan-ready preference indicates email fallback applies, when the removal notification fires, then she also receives an email with the subject "Your plan changed: {recipe_name} removed".

**FEAT-01.SPEC-018-AC-03:** Given a mid-week re-check removes three meals in a single run, when the notification fires, then Maya receives one batched notification titled "3 meals were removed from your plan" -- not three separate notifications.

**FEAT-01.SPEC-018-AC-04:** Given Maya taps the CTA on a single-meal removal notification, when she taps "Choose a replacement", then she lands on FEAT-04 (One-Tap Meal Swap) for the affected night's open slot.

**FEAT-01.SPEC-018-AC-05:** Given Maya has turned off plan-ready notifications, when a mid-week removal occurs, then she still receives this notification, since it is exempt from the ordinary preference-timing rule given its safety significance.

**FEAT-01.SPEC-018-AC-06:** Given a mid-week removal occurs during Maya's quiet hours (if any are configured elsewhere in the product), when the notification fires, then it is delivered immediately rather than held.

**FEAT-01.SPEC-018-AC-07:** Given the email channel fails to deliver after 3 retries over 6 hours, when the final retry fails, then the in-app notification stands as the delivery of record and no failure message is shown to Maya.

**FEAT-01.SPEC-018-AC-08:** Given Maya's household is deleted (FEAT-18) between the re-check and delivery, when the notification would otherwise send, then it is cancelled silently on every channel.

**FEAT-01.SPEC-018-AC-09:** Given a single meal is flagged by two rules evaluated in the same re-check run, when the notification fires, then the meal is named once, not twice.

**FEAT-01.SPEC-018-AC-10:** Given two separate hard-rule changes trigger two distinct re-check runs within moments of each other, each removing a different meal, when both complete, then Maya receives two separate notifications, one per run.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 3 (default on, off, email fallback) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
