---
document_type: feature-overview
feature_number: FEAT-01
feature_name: Household Setup & Member Profiles
feature_slug: household-setup-member-profiles
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 18
screen_count: 10
automation_count: 2
logic_rule_count: 4
integration_count: 1
notification_count: 1
---

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
