---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-001
spec_name: Account Sign-Up & Sign-In
spec_slug: account-sign-up-sign-in
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

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
