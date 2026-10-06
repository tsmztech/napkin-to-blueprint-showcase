# FEAT-20 — Onboarding / First-Run Setup

This chapter covers Onboarding / First-Run Setup, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 6 specifications carrying 100 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-20.SPEC-001 | Sign-Up & Account Creation | screen | 15 |
| FEAT-20.SPEC-002 | Onboarding Guided Sequence | screen | 27 |
| FEAT-20.SPEC-003 | Onboarding Completion Detection | automation | 14 |
| FEAT-20.SPEC-004 | Referral Attribution Capture Hand-off | automation | 14 |
| FEAT-20.SPEC-005 | Onboarding Step Sequencing & Exit-Criteria Rules | logic-rule | 21 |
| FEAT-20.SPEC-006 | Welcome Email | notification | 9 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Onboarding / First-Run Setup

## Summary

**Feature:** Onboarding / First-Run Setup
**ID:** FEAT-20
**Description:** A new freelancer is guided from signing up to sending her first proposal in one sitting: add a first client, set basic branding, and draft the first proposal.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** The decomposition checklist's Commonly Forgotten Areas require a decided first-run experience; the brief's three-month, first-paying-freelancer constraint (BRIEF.md, Constraints) makes a smooth first session essential rather than optional. Ranked Important rather than Core because the product's ongoing value does not depend on onboarding once the first project exists; phased MVP since a confusing first session directly threatens the founder's own three-month goal. [RESEARCH-INFORMED: setup burden is the category's most consistent complaint (15–25 hours for Dubsado, 20+ hours for SuiteDash; independent reviews, HIGH), so onboarding targets a ready-to-send first proposal in one short sitting] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Guided first client and project — walks the freelancer through her first setup
- Optional branding step — offered but skippable
- Guided first proposal — leads into drafting the first real proposal
- Connect payments (optional) — offers to connect her own payment account (FEAT-32) so the first deposit can be paid on the spot
- How did you hear (optional) — one question recording how she found Clientroom (FEAT-33)

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-20.SPEC-001 | Sign-Up & Account Creation | Screen | Nadia (Freelancer) | Nadia creates her Freelancer Account, which enters Active state and starts the guided sequence |
| FEAT-20.SPEC-002 | Onboarding Guided Sequence | Screen | Nadia (Freelancer), Dana (Support Operator) | The step-by-step shell that welcomes Nadia, asks the optional "how did you hear" question, hosts navigation into each guided step, shows progress, and shows the Ready state; Dana views the same progress read-only inside a logged support session |
| FEAT-20.SPEC-003 | Onboarding Completion Detection | Automation | Nadia (Freelancer) | Detects when a first client, project, and drafted proposal all exist, marks onboarding complete, and routes Nadia into the normal dashboard |
| FEAT-20.SPEC-004 | Referral Attribution Capture Hand-off | Automation | Nadia (Freelancer) | Captures the optional "how did you hear" answer and the referring-portal reference and hands both to Portal Referral Attribution (FEAT-33) for recording |
| FEAT-20.SPEC-005 | Onboarding Step Sequencing & Exit-Criteria Rules | Logic/Rule | Nadia (Freelancer) | Governs which steps are mandatory versus optional, the exit criteria for leaving onboarding, skip/resume behavior, and failure tolerance (a failed step never blocks progression) |
| FEAT-20.SPEC-006 | Welcome Email | Notification | Nadia (Freelancer) | Sends the welcome email confirming account creation, using the transactional email delivery capability |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Guided first client and project — walks the freelancer through her first setup | FEAT-20.SPEC-002, FEAT-20.SPEC-003 | The guided sequence navigates Nadia into Client & Project Management's (FEAT-01) add-client/create-project screens as a mandatory step; completion detection watches for the resulting records | Phase 2 (Explicit) |
| Optional branding step — offered but skippable | FEAT-20.SPEC-002, FEAT-20.SPEC-005 | The guided sequence navigates into Freelancer Branding's (FEAT-19) settings screen as a skippable step; the sequencing rule defines the skip and no-penalty resume behavior | Phase 2 (Explicit) |
| Guided first proposal — leads into drafting the first real proposal | FEAT-20.SPEC-002, FEAT-20.SPEC-003 | The guided sequence navigates into Proposal Creation & Sending's (FEAT-02) draft screen for the new project; completion detection watches for the resulting draft | Phase 2 (Explicit) |
| Connect payments (optional) — offers to connect her own payment account (FEAT-32) so the first deposit can be paid on the spot | FEAT-20.SPEC-002 | The guided sequence navigates into Payment Account Connection's (FEAT-32) connect screen as an optional step; the payment-processing capability contract itself belongs to FEAT-32.SPEC-002, not duplicated here (validated with no Integration spec in this feature, per the External Dependencies lens) | Phase 2 (Explicit) |
| How did you hear (optional) — one question recording how she found Clientroom (FEAT-33) | FEAT-20.SPEC-002, FEAT-20.SPEC-004 | The guided sequence presents the optional question inline on first open; the hand-off automation captures the answer and referring-portal reference and passes it to Portal Referral Attribution (FEAT-33), which owns the Referral Attribution record | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-20.SPEC-001 | Sign-Up & Account Creation | Phase 3 (Entity-Lifecycle Analysis) | The feature's Connected Entities line names Freelancer Account (create) with no covering Key Capability — sign-up itself is the missing Create operation the CRUD matrix would otherwise leave empty |
| FEAT-20.SPEC-003 | Onboarding Completion Detection | Phase 4 (Trigger-Response Analysis) | Journey step 6 ("Ready state") and the feature's own Primary Flows line ("lands in the normal dashboard once the first project exists") describe a system-side check with cross-entity, cross-feature effects (watching FEAT-01 and FEAT-02 records, then routing to FEAT-12) — a standalone Automation per the disposition rule, not an inline screen behavior |
| FEAT-20.SPEC-004 | Referral Attribution Capture Hand-off | Phase 4 (External Dependencies lens / cross-feature propagation) | The dependency map states Referral Attribution is "Created by FEAT-33 at sign-up (answer captured in FEAT-20)" — the capture-and-hand-off has cross-feature effects distinct from simply displaying the question, so it is a standalone Automation rather than folded into the screen |
| FEAT-20.SPEC-005 | Onboarding Step Sequencing & Exit-Criteria Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field states a conditional rule set (only client+project is mandatory; branding, payment connection, and billing setup are optional) plus the States field's failure-tolerance rule ("a failed step... does not block progressing") and the Primary Flows' skip/resume rule — three interacting conditional rules shared across every step, crossing the standalone-spec threshold |
| FEAT-20.SPEC-006 | Welcome Email | Phase 4 (Notification surfacing) | The Communications field names "a welcome email confirming account creation" — a message with a defined audience (Nadia), trigger (account creation), and delivery channel, which per the disposition rule must be a standalone Notification spec rather than an inline confirmation |

## Entity-Lifecycle Coverage Matrix

**Entity: Freelancer Account**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-20.SPEC-001 | Nadia signs up; the Freelancer Account record is created and immediately enters Active state | This is the entity's only Create path — the Domain Entity Inventory's inverse check confirms sign-up as the origin of every Freelancer Account |
| Read (single) | FEAT-20.SPEC-002 | The guided sequence reads the account's onboarding-progress state on every open to determine which step to resume on | Also read by FEAT-20.SPEC-003 to confirm the account exists before checking exit criteria |
| Read (list) | N/A | Exactly one Freelancer Account exists per signed-in session (there is no roster of accounts within this feature's scope; the product has no internal-staff or agency seat model per scope-boundaries.md SC-01) | — |
| Update | N/A | This feature never writes profile, business-detail, payment-terms, or notification-preference fields onto the Freelancer Account after creation — those updates belong entirely to Settings & Account Management (FEAT-21), per the dependency map ("Updated by FEAT-21"). Explicit non-goal: onboarding only creates the record and reads it to route steps | — |
| Delete/Archive | N/A | The dependency map states the Freelancer Account is "Deleted by FEAT-24" (Data Export & Account Deletion) — this feature never deletes the record. Explicit non-goal: account deletion, including any soft-delete, restore path, cascade, and retention/purge policy, is FEAT-24's lifecycle decision, not onboarding's | — |
| State Transition | FEAT-20.SPEC-001 | Created → Active happens immediately on successful sign-up, with no separate verification gate for initial creation (only a later sign-in email *change*, handled in FEAT-21, requires re-verification) | The optional "Deleted" transition is FEAT-24's, per Delete/Archive above |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client, Project | FEAT-20.SPEC-003 | Completion detection checks whether a first client and project exist under the Freelancer Account, owned and created by FEAT-01 |
| Proposal | FEAT-20.SPEC-003 | Completion detection checks whether a draft proposal exists for the new project, owned and created by FEAT-02 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia completes the sign-up form | Create the Freelancer Account and transition it to Active | Inline in triggering screen (the Create operation is the screen's core purpose) | FEAT-20.SPEC-001 |
| Freelancer Account is created | Send the welcome email confirming account creation | Standalone Notification | FEAT-20.SPEC-006 |
| Freelancer Account is created | Emit `onboarding_started` | Inline in triggering screen | FEAT-20.SPEC-001 |
| Nadia answers, or explicitly skips, "How did you hear about us?" | Capture the answer (or "unknown") and the referring-portal reference, then hand off to Portal Referral Attribution (FEAT-33) for recording | Standalone Automation | FEAT-20.SPEC-004 |
| Nadia proceeds to "Add first client and project" | Navigate into Client & Project Management's (FEAT-01) add-client/create-project screens | Cross-feature | FEAT-01 responsibility, entry point FEAT-20.SPEC-002 |
| Nadia proceeds to "Set branding" | Navigate into Freelancer Branding's (FEAT-19) settings screen, presented as skippable | Cross-feature | FEAT-19 responsibility, entry point FEAT-20.SPEC-002 |
| Nadia skips the branding step | Emit `onboarding_step_skipped`; continue to the next step with no penalty; the portal falls back to FEAT-19's clean neutral default in the meantime | Standalone Logic/Rule (skip permission), signal inline in triggering screen | FEAT-20.SPEC-005 / FEAT-20.SPEC-002 |
| A guided step's own action fails (e.g., a logo upload inside FEAT-19's step) | Allow progression to the next step regardless; the failed step remains completable later from Settings (FEAT-21), with no lost progress | Standalone Logic/Rule | FEAT-20.SPEC-005 |
| Nadia proceeds to "Connect payments" | Navigate into Payment Account Connection's (FEAT-32) connect screen, presented as optional; status shows "Ready to accept payments" once connected | Cross-feature | FEAT-32 responsibility, entry point FEAT-20.SPEC-002 |
| Nadia proceeds to "Draft the first proposal" | Navigate into Proposal Creation & Sending's (FEAT-02) draft screen for the new project | Cross-feature | FEAT-02 responsibility, entry point FEAT-20.SPEC-002 |
| Each guided step completes | Emit `onboarding_step_completed` | Inline in triggering screen | FEAT-20.SPEC-002 |
| A first client, project, and drafted proposal all exist | Mark onboarding complete, emit `onboarding_completed`, and route Nadia into the normal dashboard (FEAT-12) | Standalone Automation | FEAT-20.SPEC-003 |
| Dana opens a read-only support session on the freelancer's account (FEAT-31) | Render onboarding progress read-only, with no step controls | Inline in triggering screen (Permission-adjacent: view-only rather than hidden, per the Access Matrix's "View" row for Support Access) | FEAT-20.SPEC-002 |
| Owen or Priya attempt to reach onboarding | Nothing is shown — the Access field states onboarding is "Nadia only"; client contacts have no onboarding equivalent beyond their first magic-link login (FEAT-05) | Inline in triggering screen (Permission Denied state) | FEAT-20.SPEC-002 |
| Connectivity is lost during a guided step that creates a record (client, project) | Show a plain "this step needs a connection" message and preserve any typed-but-unsaved input; nothing pretends to succeed offline | Inline in triggering screen (Offline/Degraded state) | FEAT-20.SPEC-001 / FEAT-20.SPEC-002 |
| The freelancer account is deleted (FEAT-24) | The Freelancer Account, and with it any onboarding-progress state, ceases to exist | Cross-feature | FEAT-24 responsibility |

## Shared Context

**Shared Entities:**
- Freelancer Account — created and transitioned to Active by FEAT-20.SPEC-001; read for progress-routing by FEAT-20.SPEC-002 and for exit-criteria checks by FEAT-20.SPEC-003. No other spec in this feature writes to it (updates belong to FEAT-21, deletion to FEAT-24).

**Shared UI Patterns:**
- Guided-step chrome — FEAT-20.SPEC-002 defines the single progress indicator, "skip for now" affordance, and failure-tolerant "continue anyway" pattern that every step uses, even where the step's own content (add client/project, branding, payments, proposal) is owned by another feature's screen. Spec Writers for FEAT-01, FEAT-19, FEAT-32, and FEAT-02 should describe the chrome identically wherever their screen is reached through onboarding rather than inventing a per-step wrapper.

**Shared Validation:**
- FEAT-20.SPEC-005 is the single source of truth for which steps are mandatory versus optional, the skip/resume rule, and the failure-tolerance rule. FEAT-20.SPEC-002 (the shell) and FEAT-20.SPEC-003 (completion detection) both reference it rather than duplicating the exit-criteria logic.

## Internal Dependency Map

```
SPEC-001 (Sign-Up & Account Creation) -> [account created] -> SPEC-002 (Onboarding Guided Sequence)
SPEC-001 (Sign-Up & Account Creation) -> [account created] -> SPEC-006 (Welcome Email)
SPEC-002 (Onboarding Guided Sequence) -> [Nadia answers or skips "how did you hear"] -> SPEC-004 (Referral Attribution Capture Hand-off)
SPEC-002 (Onboarding Guided Sequence) -> [determines mandatory/optional/skip/failure behavior using] -> SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules)
SPEC-002 (Onboarding Guided Sequence) -> [after each step] -> SPEC-003 (Onboarding Completion Detection)
SPEC-003 (Onboarding Completion Detection) -> [exit criteria not yet met] -> SPEC-002 (renders next step / progress)
SPEC-003 (Onboarding Completion Detection) -> [exit criteria met] -> SPEC-002 (renders Ready state, then routes onward)
```

**Default Entry:** SPEC-001 (Sign-Up & Account Creation) — the first screen any new freelancer reaches; it is this feature's own top-level entry point (reached only once, at account creation), after which SPEC-002 becomes the active screen until onboarding completes.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-20.SPEC-002 | Inbound | FEAT-33 (Portal Referral Attribution) | A visitor follows the "Made with Clientroom" referral mark and arrives at sign-up with the referring portal already known | Visitor follows the referral mark and chooses to sign up |
| FEAT-20.SPEC-004 | Outbound | FEAT-33 (Portal Referral Attribution) | Hands off the self-reported source and referring-portal reference for Referral Attribution creation | Nadia answers, or skips, "How did you hear about us?" |
| FEAT-20.SPEC-002 | Outbound | FEAT-01 (Client & Project Management) | Guided step navigates into adding a first client and creating a project | Nadia reaches the "Add first client and project" step |
| FEAT-20.SPEC-002 | Outbound | FEAT-19 (Freelancer Branding) | Guided step navigates into branding settings, presented as skippable | Nadia reaches the "Set branding" step |
| FEAT-20.SPEC-002 | Outbound | FEAT-32 (Payment Account Connection) | Guided step navigates into connecting a payment account, presented as optional | Nadia reaches the "Connect payments" step |
| FEAT-20.SPEC-002 | Outbound | FEAT-02 (Proposal Creation & Sending) | Guided step navigates into drafting the first proposal for the new project | Nadia reaches the "Draft the first proposal" step |
| FEAT-20.SPEC-003 | Outbound | FEAT-12 (Freelancer Financial Dashboard) | Routes Nadia into the normal dashboard once onboarding's exit criteria are met | The Ready state is reached |
| FEAT-20.SPEC-002 | Inbound | FEAT-31 (Operator Support Access) | Dana views onboarding progress read-only inside a logged, time-limited support session | A support session is opened on the freelancer's account |
| FEAT-20.SPEC-006 | Outbound | FEAT-14 (Notifications — Email) | The welcome email is sent through the transactional email delivery capability | Account creation completes |
| FEAT-20.SPEC-001 | Inbound | FEAT-24 (Data Export & Account Deletion) | The Freelancer Account created here is later deleted, ending any onboarding-progress state with it | Freelancer account deletion completes |

## Non-Functional Notes

**Data volumes / growth:** Onboarding runs exactly once per new Freelancer Account, so it carries no growth concern of its own; it tracks the expected scale of a few thousand new freelancers in year one (scope-boundaries.md, SC-21).

**Responsiveness:** The First-Session Activation success metric targets a median time from sign-up to a first client, project, and drafted proposal under 15 minutes, with at least 70% of new freelancers reaching that point in their first session (success-metrics.md); each guided step should feel immediate, with no perceptible processing delay, consistent with the category's documented setup-burden complaint (15–25+ hours for Dubsado/SuiteDash) that this feature exists to avoid. The Payment Readiness metric further expects connecting payments to take under 5 minutes when Nadia chooses that optional step (success-metrics.md).

**Data sensitivity / privacy:** The Freelancer Account created here holds the freelancer's own personal data — name and sign-in email — treated as GDPR-class personal data, exportable and deletable on request (assumptions-constraints.md, ASMP-23, ASMP-24). The optional "how did you hear" answer captured for Referral Attribution is low personal data used only in aggregate, and the referring freelancer is never told who signed up (feature-dependency-map.md, Referral Attribution, Data Sensitivity).

**Compliance flags:** Any operator access to onboarding progress is read-only, and every such support session is announced to the freelancer by email and listed in her trail (assumptions-constraints.md, ASMP-23; feature-dependency-map.md, XBR-29). Because onboarding sends the welcome email, it is covered by the delivery-visibility expectation that failed transactional emails are surfaced to the freelancer within minutes rather than lost (ASMP-26).

## Non-Goals

- **A configurable workflow, form, or automation builder for the onboarding sequence** — Excluded per scope-boundaries.md (SC-11): the product ships fixed, sensible onboarding behavior rather than a customizable step builder, in line with the brief's promise to avoid the category's configuration-driven setup burden.
- **Bulk import of clients, projects, or invoice history during onboarding** — Excluded per scope-boundaries.md (SC-19): the guided "first client and project" step adds one client by hand; importing historical records is out of scope everywhere in the product, including here, since imported records were never accepted or sent through Clientroom.
- **Team, agency, or multi-seat onboarding (inviting collaborators during setup)** — Excluded per scope-boundaries.md (SC-01): the product has no internal-staff or agency seat model, so onboarding has no step for inviting anyone beyond the single freelancer signing up.
- **Skipping the first client, project, or proposal-draft step** — Only branding, payment connection, and full billing setup are optional; adding a first client and project and starting the first proposal are the feature's mandatory exit criteria (product-features.md, Validation & Limits). This is a deliberate design decision, not an oversight: without it, "onboarding complete" would have no reliable meaning for the First-Session Activation metric.
- **A separate retention or purge policy for onboarding-progress data** — Adjacency exclusion, surfaced by the CRUD matrix: onboarding-progress state lives on the Freelancer Account itself and is retired only when the account is deleted (FEAT-24); no independent expiry or archival timer applies to a one-time, per-account setup sequence.



# Screen Spec: Sign-Up & Account Creation

## Overview

**Name:** Sign-Up & Account Creation
**ID:** FEAT-20.SPEC-001
**Type:** Screen
**Purpose:** A new freelancer creates her Freelancer Account, which is created and immediately transitioned to Active state, starting the guided onboarding sequence.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- Capturing the minimum data needed to create a Freelancer Account: name, sign-in email, and a sign-in credential
- Carrying forward the referring-portal reference when the visitor arrived via a "Made with Clientroom" mark (FEAT-33), and persisting it on the new Freelancer Account's onboarding-progress state at creation so it survives across sessions
- Initialising the Freelancer Account's onboarding-progress state to the defaults defined in FEAT-20.SPEC-005 (Defaults and Derivations) at the moment of creation
- Creating the Freelancer Account and transitioning it to Active state on successful submission
- Emitting `onboarding_started` and handing off into the guided sequence (FEAT-20.SPEC-002)
- Preserving entered data and offering retry on a failed save, including while offline

**Non-Goals:**
- Capturing business details, payment terms, or a time zone -- those fields belong to the Freelancer Account's later completion in Settings & Account Management (FEAT-21), not to sign-up; onboarding creates the record with only its two required fields (name, sign-in email) and reads it thereafter
- Asking "How did you hear about us?" -- that optional question is asked inside the guided sequence (FEAT-20.SPEC-002), not on this screen, per the Brief's Side-Effect Inventory
- Client-contact sign-in -- Owen and Priya reach their portals through a magic link (FEAT-05.SPEC-002), never through this form; this screen exists only to create a new Freelancer Account
- A separate email-verification gate before the account becomes Active -- per the Entity-Lifecycle Coverage Matrix, "Created → Active happens immediately on successful sign-up, with no separate verification gate for initial creation"; only a later sign-in email *change* (FEAT-21) requires re-verification

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-33 (Portal Referral Attribution) "Made with Clientroom" mark | Visitor follows the mark on a client-facing portal page or email and chooses to sign up | The referring Freelancer Account reference, carried silently through the form to account creation, where it is persisted on the account's onboarding-progress state and later handed off by FEAT-20.SPEC-004 |
| External marketing entry (organic, direct, search) | Visitor reaches the product's public sign-up entry with no referral context | None -- referring-portal reference is absent; FEAT-20.SPEC-004 records the source as unknown unless answered later |

## Access and Visibility

This screen has no signed-in state of its own -- it is the entry point that creates a session, so "Can Act" describes who may complete it rather than a permission grant on an existing record.

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Unauthenticated visitor | Yes | Yes -- may create a new Freelancer Account | -- |
| Nadia (Freelancer, already signed in) | Yes, briefly | No -- an already-authenticated freelancer is redirected before the form renders | Redirected to her post-sign-in landing destination (FEAT-20.SPEC-005 Rule R-08: her current onboarding step in FEAT-20.SPEC-002 while onboarding is In Progress, otherwise the dashboard, FEAT-12) with no message needed, since nothing was denied -- she already has an account |
| Owen (Client Primary Contact, signed in to a portal session) | Yes | Yes -- may create a separate, unrelated Freelancer Account of her own (the referral growth path in FEAT-33's happy path: "a client contact who is also a freelancer... follows the mark -> signs up"); doing so never affects his existing portal access | -- |
| Priya (Client Reviewer Contact, signed in to a portal session) | Yes | Yes -- identical to Owen's case above | -- |
| Dana (Support Operator) | Yes | No -- the operator identity is provisioned separately from any product sign-up path, per BRIEF.md's "not a product role" | This screen has no awareness of the operator identity; completing it would simply create an ordinary Freelancer Account, which is never how Dana's access is granted (FEAT-31 owns operator provisioning) |
| Expired session | Yes | Yes -- this screen requires no existing session, so an expired session has no effect on reaching or completing it | -- |

## Layout and Content

**Header:** Product wordmark, and a "Sign in" link (for an existing freelancer who reached this page by mistake) at the top right.

**Body:** A single-column form with the following fields, in order:
- Full name (text input, required)
- Sign-in email (text input, required)
- Password (masked text input, required)
- Confirm password (masked text input, required)
- Terms of Service and Privacy acknowledgment (checkbox, required; the label contains two inline links, "Terms of Service" and "Privacy Policy", each opening its document)
- "Create my account" button, below the form fields, full width

All fields use one consistent input treatment platform-wide. Required-field indicators are shown on every field, since every field here is required.

**Footer:** A single line: "Already have an account? Sign in" linking to the sign-in screen.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width, as described above; the "Create my account" button remains directly below the form fields.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Full name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Full name input | Blur (empty) | Triggers field validation | Error state on field | "Your name is required" below field |
| Sign-in email input | Blur | Validates email format and uniqueness | Error state if invalid or already registered | "Enter a valid email address" or "An account already exists for this email. Sign in instead." with a link to sign-in |
| Password input | Type | Captures input, evaluates minimum strength | Field shows entered characters (masked) | Inline strength/requirement hint below the field |
| Confirm password input | Blur | Compares against Password value | Error state if mismatched | "Passwords don't match" |
| Terms checkbox | Tap | Toggles acknowledgment | Checkbox shows checked/unchecked | -- |
| "Sign in" link (header) | Tap | Navigate to the sign-in screen | Screen closes; entered values are discarded, with no confirmation (no record exists yet) | Standard navigation transition |
| "Sign in" link (footer, "Already have an account? Sign in") | Tap | Navigate to the sign-in screen (identical to the header link) | Screen closes; entered values are discarded, with no confirmation | Standard navigation transition |
| "Terms of Service" link (inside the checkbox label) | Tap | Opens the Terms of Service document in a new browser tab, leaving this form untouched | No change to any field or to the checkbox state | The document opens in the new tab; closing it returns Nadia to the form with every entered value intact |
| "Privacy Policy" link (inside the checkbox label) | Tap | Opens the Privacy Policy document in a new browser tab, leaving this form untouched | No change to any field or to the checkbox state | Same as the Terms of Service link |
| Error banner "Retry" button | Tap | Re-submits the same validated field values through steps 2-4 of the "Create my account" action (create the Freelancer Account, emit `onboarding_started`, navigate to FEAT-20.SPEC-002); the referring-portal reference is re-sent unchanged | Banner is replaced by the Creating state (button loading, fields disabled) | Success: navigates into FEAT-20.SPEC-002. Failure: the banner returns with the failure message and the Retry button |
| "Create my account" button | Tap | 1. Validate all fields. 2. If valid, create the Freelancer Account and transition it to Active (which fires FEAT-20.SPEC-006, Welcome Email, independently of this screen's navigation). 3. Emit `onboarding_started`. 4. Navigate to FEAT-20.SPEC-002. | Button shows loading state during creation | Success: navigates directly into FEAT-20.SPEC-002 (Onboarding Guided Sequence), which opens on its Welcome state. Failure: inline error messages; button returns to normal state. |
| "Create my account" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Full name -> Sign-in email -> Password -> Confirm password -> Terms checkbox -> Terms of Service link -> Privacy Policy link -> Create my account -> Sign in (footer link). When the Error banner is showing, focus moves to the banner and its Retry button precedes the first field.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Creation feedback:** On failure, an error banner is announced and focus moves to the first field in error (or to the banner itself for a non-field error such as a connectivity failure).
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-------------------|------------------|
| Empty (default) | All form fields empty, "Create my account" enabled | Screen first opens | User begins typing in any field |
| Filling | Form fields contain user input | User types in any field | User taps "Create my account" or navigates away |
| Validation Error | Failed fields highlighted with error messages below them | Client-side validation fails on blur or submit | User corrects the field and re-triggers validation |
| Creating | Button shows loading state, form fields disabled | Validation passes and account creation begins | Creation completes or fails |
| Error | Error banner at top of form with a retry option | Account creation fails (e.g., a transient failure after validation passed) | User taps Retry or corrects the field named in the error and resubmits |
| Offline/Degraded | Banner "This step needs a connection. Your details are saved here and will be submitted once you're back online." at the top; form remains editable and entered values are preserved | Connectivity is lost while the screen is open or while creation is in progress | Connectivity is restored -- the user taps "Create my account" again to submit (no silent background retry, since account creation must never appear to succeed without the freelancer's own confirming action) |

## Validation Rules

| Field | Condition | When Checked | Error Message |
|-------|-----------|-----------------|------------------|
| Full name | Required, non-empty, max 100 characters | On blur | "Your name is required" / "Name must be 100 characters or fewer" |
| Sign-in email | Required; valid email format; must not already belong to a Freelancer Account | On blur and on submit | "Enter a valid email address" / "An account already exists for this email. Sign in instead." |
| Password | Required; minimum length and a mix of character types | On blur | "Choose a stronger password" (with an inline hint of the specific unmet requirement) |
| Confirm password | Must exactly match Password | On blur and on submit | "Passwords don't match" |
| Terms acknowledgment | Must be checked | On submit | "You must accept the Terms of Service and Privacy Policy to continue" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|---------------------------------------|
| "Sign in" link (header or footer) tap | Sign-in screen | -- (outside this feature's scope; owned by the product's baseline authentication surface) |
| "Terms of Service" link tap | Terms of Service document, in a new browser tab | -- (the product's legal-document surface, outside this feature's scope) |
| "Privacy Policy" link tap | Privacy Policy document, in a new browser tab | -- (the product's legal-document surface, outside this feature's scope) |
| Error banner "Retry" success | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | -- |
| Successful account creation | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | -- |

## Data Model

**Creates:** Freelancer Account -- `name` and sign-in `email` set from form input; a sign-in credential is set (never displayed back to the freelancer or visible to the operator); `status` set to Active immediately, with no intermediate unverified state. In the same atomic step, the account's onboarding-progress state is initialised to the FEAT-20.SPEC-005 defaults (current step "How did you hear", status In Progress, empty completed/skipped/failed sets, question unresolved, Ready screen unacknowledged, welcome-email warning not dismissed) and `onboarding_referring_portal_ref` is set to the referring-portal reference carried from a FEAT-33 entry, or to absent when there was none. All other Freelancer Account fields (business details, payment terms, time zone, notification preferences) remain unset at this point and are completed later through Settings & Account Management (FEAT-21).
**Reads:** None -- this is a creation screen.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Account creation and the Active state transition happen as one atomic step -- there is no intermediate "pending" Freelancer Account visible anywhere in the product.
- The referring-portal reference carried from a FEAT-33 entry is retained in the form's context through any creation retry, and is persisted on the new Freelancer Account's onboarding-progress state (`onboarding_referring_portal_ref`) as part of the same atomic creation. From that point it lives on the account, not in the browser session, so closing the browser before the "How did you hear" moment never loses it; FEAT-20.SPEC-004 reads it from the account when Nadia answers or skips (or when onboarding completes first).
- This screen is the sole initialiser of the onboarding-progress defaults: the defaults in FEAT-20.SPEC-005 (Defaults and Derivations) are applied here, at creation, and nowhere else.
- `onboarding_started` is emitted exactly once, at the moment the Freelancer Account is created -- never re-emitted on a later visit to onboarding.
- On successful creation, the welcome email (FEAT-20.SPEC-006) is triggered independently of this screen's own navigation -- a slow or failed email send never blocks or delays entry into FEAT-20.SPEC-002.

## Edge Cases

- **Two visitors submit the same email at effectively the same time** -- The email-uniqueness check is re-verified authoritatively at the moment of commit; the first submission to commit succeeds, and the second is rejected with "An account already exists for this email. Sign in instead." even if both passed the earlier blur-time check.
- **User navigates away mid-form** -- No confirmation dialog is shown (no record exists yet to lose); returning to the sign-up entry point starts with an empty form.
- **User taps "Create my account" twice rapidly** -- Second tap is ignored while the first creation attempt is in progress (button in loading state).
- **Creation succeeds but the subsequent navigation is interrupted (e.g., the browser closes)** -- The Freelancer Account already exists and is Active; the freelancer's next sign-in resumes directly at her current onboarding step (FEAT-20.SPEC-002), which reads onboarding-progress state from the account rather than assuming this screen's navigation completed.
- **User arrives here already signed in** -- Handled by the Access and Visibility redirect above; no form is shown.
- **Connectivity is lost after the user has typed but before submitting** -- The Offline/Degraded state applies; typed values are preserved and the user submits once reconnected, with no automatic background submission.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Navigation (outbound) | Successful account creation navigates directly into the guided sequence's Welcome state |
| FEAT-20.SPEC-006 (Welcome Email) | Triggers (outbound) | Account creation triggers the welcome email confirming account creation |
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | References (outbound) | The referring-portal reference carried from a FEAT-33 entry is passed through this screen for later hand-off |
| FEAT-33 (Portal Referral Attribution) | Navigation (inbound) | A visitor following the "Made with Clientroom" mark arrives here with the referring portal known |
| FEAT-24 (Data Export & Account Deletion) | References (inbound) | The Freelancer Account created here is the same record FEAT-24 later deletes, ending any onboarding-progress state with it |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|-------------------|----------------------|
| onboarding_started | referring-portal reference present (yes/no) | Freelancer Account is created and transitioned to Active | supports success-metrics.md: "First-Session Activation" |
| account_creation_failed | failure reason (validation / duplicate-email / connectivity) | Creation attempt does not result in an Active Freelancer Account | N/A -- no Stage 2 metric measures failed sign-up attempts directly; retained so first-session friction is observable rather than invisible |

## Acceptance Criteria

**FEAT-20.SPEC-001-AC-01:** Given a new visitor is on the Sign-Up screen, when she fills in her name, a unique sign-in email, matching passwords, accepts the Terms, and taps "Create my account", then her Freelancer Account is created in Active state, `onboarding_started` is emitted, and she is taken directly into FEAT-20.SPEC-002 (Onboarding Guided Sequence).

**FEAT-20.SPEC-001-AC-02:** Given a visitor is on the Sign-Up screen, when she taps "Create my account" with the name field empty, then the name field shows "Your name is required" and creation does not proceed.

**FEAT-20.SPEC-001-AC-03:** Given a visitor enters an email that already belongs to an existing Freelancer Account, when she blurs the email field, then she sees "An account already exists for this email. Sign in instead." with a link to sign-in.

**FEAT-20.SPEC-001-AC-04:** Given a visitor enters mismatched values in Password and Confirm Password, when she blurs the Confirm Password field, then she sees "Passwords don't match".

**FEAT-20.SPEC-001-AC-05:** Given a visitor fills the form correctly but leaves the Terms checkbox unchecked, when she taps "Create my account", then she sees "You must accept the Terms of Service and Privacy Policy to continue" and no account is created.

**FEAT-20.SPEC-001-AC-06:** Given Nadia (already signed in) navigates directly to the Sign-Up screen's URL, when the screen would otherwise render, then she is redirected to her current onboarding step or dashboard instead, with no form shown.

**FEAT-20.SPEC-001-AC-07:** Given a client contact who is also a freelancer follows the "Made with Clientroom" mark from a portal she was viewing (FEAT-33), when she completes this form, then a new Freelancer Account is created with the referring-portal reference carried forward, unrelated to her existing client-contact access.

**FEAT-20.SPEC-001-AC-08:** Given a visitor loses connectivity while filling the form, when she attempts to submit, then the banner "This step needs a connection. Your details are saved here and will be submitted once you're back online." appears and her typed values remain in the form.

**FEAT-20.SPEC-001-AC-09:** Given two visitors submit the same email at effectively the same time, when the second submission's account creation is committed, then it is rejected with "An account already exists for this email. Sign in instead." even though it may have passed the earlier field-level check.

**FEAT-20.SPEC-001-AC-10:** Given a visitor taps "Create my account" while a previous submission from the same tap is still processing, when she taps a second time, then the second tap has no effect and the button remains in its loading state.

**FEAT-20.SPEC-001-AC-11:** Given Dana's operator identity has no product sign-up path, when anyone attempts to provision operator access through this screen, then the attempt is undefined here -- it would only ever create an ordinary Freelancer Account, never operator access, since FEAT-31 owns that provisioning separately.

**FEAT-20.SPEC-001-AC-12:** Given a visitor arrived via a "Made with Clientroom" mark and completes the form, when the Freelancer Account is created, then `onboarding_referring_portal_ref` holds that reference on the account, the onboarding-progress state holds the FEAT-20.SPEC-005 defaults, and if Nadia closes the browser before reaching the "How did you hear" question the reference is still present on the account at her next sign-in.

**FEAT-20.SPEC-001-AC-13:** Given a visitor has typed values into the form, when she taps the "Terms of Service" or "Privacy Policy" link in the checkbox label, then that document opens in a new browser tab and, on returning to the form, every entered value and the checkbox state are unchanged.

**FEAT-20.SPEC-001-AC-14:** Given the Error banner is showing after a transient creation failure, when she taps its "Retry" button, then the same validated values (and the same referring-portal reference) are re-submitted, and on success she is taken into FEAT-20.SPEC-002 with `onboarding_started` emitted exactly once.

**FEAT-20.SPEC-001-AC-15:** Given a visitor is on the Sign-Up screen, when she taps "Already have an account? Sign in" in the footer, then she is taken to the sign-in screen, exactly as with the header "Sign in" link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Interactions | 13 | 13 |
| States | 6 (empty, filling, validation error, creating, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Onboarding Guided Sequence

## Overview

**Name:** Onboarding Guided Sequence
**ID:** FEAT-20.SPEC-002
**Type:** Screen
**Purpose:** The step-by-step shell that welcomes Nadia, asks the optional "how did you hear" question, hosts navigation into each guided step, shows progress, shows the Ready state (with a "Go to your dashboard" button) once onboarding's exit criteria are met, shows the welcome-email delivery warning if that email could not be delivered; Dana views the same progress read-only inside a logged support session.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- The Welcome moment and the optional "How did you hear about us?" question, shown on every open until Nadia answers or skips it (then never again)
- The single progress indicator, "skip for now" affordance, and failure-tolerant "continue anyway" pattern shared by every step, per the Brief's Shared Context (Shared UI Patterns)
- Hosting navigation into each guided step's own screen (owned by FEAT-01, FEAT-19, FEAT-32, FEAT-02) and returning from it
- Displaying the Ready state once FEAT-20.SPEC-003 reports onboarding complete, until Nadia taps its "Go to your dashboard" button; after that, redirecting to the dashboard (FEAT-12) on any open
- Surfacing the welcome-email delivery warning defined by FEAT-20.SPEC-006 (surface, wording, dismissal)
- Re-triggering FEAT-20.SPEC-003 on every open while onboarding is In Progress, and offering "Check again" when a completion check could not finish
- Dana's read-only view of the same progress inside a logged support session (FEAT-31)

**Non-Goals:**
- Which steps are mandatory versus optional, skip/resume behavior, and failure-tolerance rules -- owned by FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules); this shell reads and applies that spec's rules rather than redefining them
- Determining when onboarding is complete -- owned by FEAT-20.SPEC-003 (Onboarding Completion Detection); this shell only renders whatever state that automation reports. Routing is this shell's job alone: FEAT-20.SPEC-003 signals, it never navigates
- The content of each guided step itself (adding a client and project, setting branding, connecting payments, drafting a proposal) -- each is owned by its own feature's screen (FEAT-01, FEAT-19, FEAT-32, FEAT-02 respectively); this shell provides only the shared chrome around them
- A configurable workflow or step builder -- excluded per scope-boundaries.md SC-11: the product ships one fixed, sensible onboarding sequence rather than a customizable one

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Successful account creation | New Freelancer Account, no onboarding-progress state yet -- opens on the Welcome state |
| Direct return (sign-in landing, bookmark, browser reopen) and FEAT-20.SPEC-006's "Continue setup" email link | Nadia signs in or opens the location (post-sign-in landing rule, FEAT-20.SPEC-005 R-08) | Existing onboarding-progress state is read. In Progress: the shell resumes on the step it reflects (the Welcome state if the "How did you hear" question is still unresolved). Complete and Ready not yet acknowledged: the Ready state. Complete and acknowledged: no shell is rendered; Nadia is redirected to FEAT-12 with no message |
| FEAT-01, FEAT-19, FEAT-32, FEAT-02's own onboarding-context screens | Nadia completes or exits a guided step's own screen | The completed or exited step's outcome, so this shell can advance, mark skipped, or mark failed-but-continuable |
| FEAT-31 (Operator Support Access) | Dana opens a read-only support session on the freelancer's account | Session is read-only; no step controls render; the welcome-email warning, if applicable, renders without its buttons |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Nadia (Freelancer) | Full screen | All actions: answer or skip "how did you hear," enter any step, skip a skippable step, continue past a failed step | -- |
| Owen (Client Primary Contact) | No | No | This location is not served to a portal session. Owen sees a plain page titled "This page isn't available" with the text "That page isn't part of your portal." and a "Back to your portal" button that leads to his portal home (FEAT-05). No onboarding data is loaded (the Access field states onboarding is "Nadia only") |
| Priya (Client Reviewer Contact) | No | No | Identical to Owen: the "This page isn't available" page with "That page isn't part of your portal." and a "Back to your portal" button leading to her portal home (FEAT-05) |
| Dana (Support Operator) | Inside an open support session on this account: full progress view, no step controls. Outside an open support session: No | No -- view-only, per the Access Matrix's "View" row for Support Access | Inside a session, step-entry, "Skip for now," "Try again," "Check again" and "continue anyway" controls and the warning's buttons are not rendered. Outside a session, this location is not served to her: she sees a plain page titled "This page isn't available" with the text "Open a support session on a freelancer's account to view their setup progress." and a "Go to support access" button leading to FEAT-31; no freelancer data is loaded |
| Unauthenticated | No | No | Redirected to the sign-in screen; onboarding requires a Freelancer Account |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any in-progress guided-step input (e.g., a partially filled branding form reached from here) is preserved by that step's own screen and restored after re-authentication succeeds |

## Layout and Content

**Header:** A single progress indicator showing all steps in order -- "How did you hear" (shown until answered or skipped), "Add your first client and project," "Set your branding" (marked Optional), "Connect payments" (marked Optional), "Draft your first proposal" -- with the current step highlighted and completed steps marked done. Dana's read-only session shows the identical indicator with no highlighting affordance for interaction.

**Notice area (top of Body, above the panel):** Zero or more of the following notices, each visible only when its condition holds:
- *Welcome-email delivery warning:* "We couldn't deliver your welcome email to {sign_in_email}. If that address is wrong, you can correct it in Settings." with a "Go to Settings" button and a "Dismiss" link. Shown when FEAT-20.SPEC-006 reports its retries exhausted for this account, the sign-in email has not since been changed, and onboarding_welcome_warning_dismissed is false. Shown in the In Progress, Welcome and Ready states. In Dana's session the notice text shows with neither button nor link.
- *Completion-check notice:* "We couldn't confirm your setup just now." with a "Check again" button. Shown only after FEAT-20.SPEC-003 reports that a check could not finish.

**Body:** A single content area that hosts either:
- The Welcome message and the "How did you hear about us?" question (shown on every open until answered or skipped): a short welcome line, one text-or-selection input for the answer, a "Skip" link, and a "Continue" button.
- The current step's own launch panel: the step's name, one line describing what it accomplishes, a "Not finished" tag if its last attempt failed (mandatory steps only), and an "Add first client and project" / "Set branding" / "Connect payments" / "Draft the first proposal" button that navigates into that step's owning feature's screen. Skippable steps (branding, payments) additionally show a "Skip for now" link beside the button.
- The Ready state: a confirmation message that the first client, project, and drafted proposal all exist, and a "Go to your dashboard" button. The shell does not route automatically; Nadia stays on the Ready state until she taps the button.

**Footer:** None -- all actions live in the body panel for the current state.

### Responsive Behavior

- **Compact breakpoint:** Progress indicator collapses to a compact form showing only "Step {n} of {total}" plus the current step's name, with the full step list reachable by tapping it; body panel remains single-column, full width.
- **Medium size class and above:** Full progress indicator with all step names visible across the header; body panel capped at a consistent platform-wide content width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| "How did you hear" input | Type or select | Captures the answer | Field shows entered value | Standard input focus state |
| "How did you hear" Continue button | Tap | Triggers FEAT-20.SPEC-004 with the entered answer (FEAT-20.SPEC-004 alone sets onboarding_how_did_you_hear_resolved; this screen never writes it), then advances to the first mandatory step without waiting for the hand-off | Screen moves to the next step's launch panel | Brief transition; no separate confirmation needed |
| "How did you hear" Skip link | Tap | Triggers FEAT-20.SPEC-004 with "unknown" as the answer (this screen never writes onboarding_how_did_you_hear_resolved), then advances without waiting for the hand-off | Screen moves to the next step's launch panel | -- |
| Progress indicator (any completed or current step) | Tap | Navigates to that step's launch panel (Nadia only) | Body panel switches to the tapped step | -- |
| Progress indicator (Dana's session) | Tap | No action -- display only in a read-only session | None | -- |
| "Add first client and project" button | Tap | Navigates to FEAT-01.SPEC-001 (Add Client), carrying the onboarding context | This shell is left in place behind the navigation; returns here on completion or exit | Standard navigation transition |
| "Set branding" button | Tap | Navigates to FEAT-19.SPEC-001 (Branding Settings), carrying the onboarding context | Same as above | Standard navigation transition |
| "Set branding" -> "Skip for now" link | Tap | Applies FEAT-20.SPEC-005's skip rule; emits `onboarding_step_skipped`; advances to the next step | Body panel switches to the next step's launch panel | Brief confirmation: "You can set this up anytime from Settings." |
| "Connect payments" button | Tap | Navigates to FEAT-32.SPEC-001 (Payment Connection Screen), carrying the onboarding context | Same as branding/client above | Standard navigation transition |
| "Connect payments" -> "Skip for now" link | Tap | Applies FEAT-20.SPEC-005's skip rule; emits `onboarding_step_skipped`; advances to the next step | Body panel switches to the next step's launch panel | Brief confirmation: "You can connect payments anytime from Settings." |
| "Draft the first proposal" button | Tap | Navigates to FEAT-02.SPEC-001 (Proposal Draft Editor) for the new project, carrying the onboarding context | Same as above | Standard navigation transition |
| Returning from a step's own screen (completed) | Automatic, on return | Emits `onboarding_step_completed`; triggers FEAT-20.SPEC-003 to re-check exit criteria; advances to the next step or the Ready state | Progress indicator marks the step done | Brief confirmation on the step just completed |
| Returning from an optional step's own screen (failed action, e.g., a logo upload failure inside FEAT-19) | Automatic, on return with a failure signal | Applies FEAT-20.SPEC-005's Rule R-04 for optional steps: adds the step to onboarding_failed_incomplete_steps, emits `onboarding_step_failed_continued`, allows progression | Body panel advances to the next step; the failed step is marked "Finish later from Settings" rather than done | Brief message: "We'll let you pick this back up from Settings later." |
| Returning from a mandatory step's own screen (failed action, e.g., client or project creation, or proposal drafting, failed) | Automatic, on return with a failure signal | Applies FEAT-20.SPEC-005's Rule R-04 for mandatory steps: no progression, nothing recorded as continued-past; the step stays current | The same step's launch panel shows a "Not finished" tag, its button relabelled "Try again" (same destination as the original button), and no "Skip for now" or "continue anyway" | Message: "That didn't finish saving. Try again when you're ready." Nadia may still leave onboarding at any time |
| "Check again" button (Completion-check notice) | Tap | Re-triggers FEAT-20.SPEC-003 | Notice replaced by a loading indicator; on a successful check the notice is removed and the shell renders the outcome (Ready, or the current step) | If the check fails again the notice returns with "We couldn't confirm your setup just now." |
| Shell opens while onboarding is In Progress | Automatic, on open (after the progress read succeeds) | Triggers FEAT-20.SPEC-003 to re-check the exit criteria | If the check reports complete, the shell renders the Ready state | None unless the check cannot finish (Completion-check notice) |
| Ready state "Go to your dashboard" button | Tap | Sets onboarding_ready_acknowledged to true, then navigates to FEAT-12 (Freelancer Financial Dashboard) | Onboarding shell is exited; later opens of this location redirect to FEAT-12 | Standard navigation transition |
| Welcome-email warning "Go to Settings" button | Tap | Navigates to FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) so Nadia can correct the address | Shell is left in place behind the navigation; the warning stays until the address is changed or the warning is dismissed | Standard navigation transition |
| Welcome-email warning "Dismiss" link | Tap | Sets onboarding_welcome_warning_dismissed to true | Warning is removed and never shown again | -- |
| Leaving to any other feature (dashboard, clients, Settings) while In Progress | Nadia navigates through the product's normal navigation | Nothing is blocked (FEAT-20.SPEC-005 Rule R-08); onboarding-progress state is left unchanged | This shell is left; the next sign-in lands here again at the current step | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Progress indicator (as a landmark, not a tab stop for Dana's read-only session) -> body panel content, top to bottom -> primary action button -> secondary "Skip for now" link where present.
- **Step-change announcements:** When the body panel switches to a new step (by completion, skip, or manual navigation), the new step's name is announced to assistive technology.
- **Confirmation announcements:** Brief confirmations (skip, continue-anyway) are announced as they appear.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-------------------|------------------|
| Loading | Progress indicator skeleton shown, body panel empty | Screen opens and onboarding-progress state is being read from the Freelancer Account | The read completes -- shell renders Welcome, In Progress, or Ready based on the result |
| Load Error | Banner "We couldn't load your setup progress." with a Retry button; no step panel shown | Reading onboarding-progress state fails | User taps Retry and the read succeeds |
| Welcome | Welcome message and "How did you hear" question shown | onboarding_current_step is "How did you hear", onboarding_how_did_you_hear_resolved is false and onboarding_status is In Progress (FEAT-20.SPEC-005 R-06) -- on first open after sign-up and on every return until she answers or skips (closing the browser without answering leaves it unresolved) | Nadia answers or skips the question |
| In Progress -- Step {n} | Current step's launch panel shown, progress indicator reflects position | A step is active and not yet completed, skipped, or continued-past | The step is completed, skipped (if skippable), or continued past after a failure |
| Completion Check Pending | The current step's panel plus the Completion-check notice with "Check again" | FEAT-20.SPEC-003 reports that a check could not finish (read failure), on a step return or on shell open | A later check (via "Check again", the next shell open, or the next step completion) finishes: the notice is removed and the shell shows Ready or the current step |
| Ready | Confirmation message and "Go to your dashboard" button; no step controls | onboarding_status is Complete and onboarding_ready_acknowledged is false | Nadia taps "Go to your dashboard" (the only exit; there is no automatic reroute) |
| Redirected (Complete and acknowledged) | No shell is rendered | onboarding_status is Complete and onboarding_ready_acknowledged is true, on any open of this location | Nadia lands on FEAT-12 with no message |
| Dana's Read-Only View | Identical progress indicator and current-step name, with no interactive controls | A support session (FEAT-31) is opened on this freelancer's account | The support session closes |
| Offline/Degraded | Banner "This step needs a connection." at the top of the body panel; the shell's own navigation and progress reading remain available (no record is created by this shell itself), but any step reached from here that creates a record shows its own offline behavior | Connectivity is lost while this shell is open | Connectivity is restored |

## Validation Rules

Validation of what is skippable, mandatory, and failure-tolerant is governed by FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules). See that spec for the complete rule set. This screen applies those rules to decide which "Skip for now" links appear and whether a failed step blocks progression.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|---------------------------------------|
| "Add first client and project" | FEAT-01.SPEC-001 (Add Client) | FEAT-01 (Client & Project Management) |
| "Set branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| "Connect payments" | FEAT-32.SPEC-001 (Payment Connection Screen) | FEAT-32 (Payment Account Connection) |
| "Draft the first proposal" | FEAT-02.SPEC-001 (Proposal Draft Editor) | FEAT-02 (Proposal Creation & Sending) |
| Ready state "Go to your dashboard" | Freelancer Financial Dashboard | FEAT-12 (Freelancer Financial Dashboard) |
| Any open when Complete and acknowledged (redirect) | Freelancer Financial Dashboard | FEAT-12 (Freelancer Financial Dashboard) |
| Welcome-email warning "Go to Settings" | FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | FEAT-21 (Settings & Account Management) |
| Owen or Priya reaching this location ("Back to your portal") | Portal home | FEAT-05 (Client Portal Access) |
| Dana outside a support session ("Go to support access") | Support access entry | FEAT-31 (Operator Support Access) |

## Data Model

**Creates:** None directly -- record creation happens in the destination features' own screens (Client, Project, Branding Profile, Payment Account Connection, Proposal).
**Reads:** Freelancer Account -- its onboarding-progress state (which step is current, which steps are completed or skipped, whether any step is marked "finish later", onboarding_how_did_you_hear_resolved, onboarding_ready_acknowledged, onboarding_welcome_warning_dismissed), read on every open to determine which state to render. Also reads the welcome email's delivery outcome from the transactional email delivery status (FEAT-14.SPEC-001 / FEAT-14.SPEC-003) and the current sign-in email (to hide the warning once it has changed).
**Updates:** Freelancer Account -- its onboarding-progress state only, advanced as steps complete, are skipped, or are continued past after failure, plus onboarding_ready_acknowledged (on "Go to your dashboard") and onboarding_welcome_warning_dismissed (on "Dismiss"). onboarding_how_did_you_hear_resolved is read here but never written by this screen: FEAT-20.SPEC-004 is its only writer. No other Freelancer Account field (name, email, business details, payment terms, notification preferences) is written by this screen; those updates belong entirely to FEAT-21.
**Deletes:** None.

## Business Rules

- Step sequencing, mandatory/optional status, skip behavior, and failure tolerance are all governed by FEAT-20.SPEC-005 -- this shell never defines its own version of these rules.
- Completion is determined exclusively by FEAT-20.SPEC-003; this shell renders whatever state that automation reports and never independently decides onboarding is complete.
- The "How did you hear" question is resolved exactly once: it is shown on every open until Nadia answers or skips it, and onboarding_how_did_you_hear_resolved is set true by FEAT-20.SPEC-004 after its hand-off attempt on answer or skip (this screen only triggers it) -- never on render. A Nadia who saw it and left is asked again on return; a Nadia who answered or skipped is never asked again (FEAT-20.SPEC-005 R-06). If onboarding completes first, the question is auto-resolved as "unknown" and never shown (R-09).
- Ready-state model: the shell never auto-routes. On reaching Complete, the Ready state stays until Nadia taps "Go to your dashboard", which records onboarding_ready_acknowledged; any open of this location after that redirects to FEAT-12. FEAT-20.SPEC-003 only signals completion and never navigates.
- No navigation gate: while onboarding is In Progress Nadia can open any other feature freely; the shell is her post-sign-in landing destination, not a barrier (FEAT-20.SPEC-005 R-08). Onboarding-progress state is unchanged by leaving.
- The welcome-email delivery warning is the account-level variant of the FEAT-14.SPEC-006 delivery-failure warning, shown here because no project exists yet on which to show it. It disappears when Nadia dismisses it or changes her sign-in email; after the Ready state is acknowledged it is no longer surfaced, since her account is fully usable and she has already signed in with that address.
- Dana's session never shows step-entry controls, "Skip for now," or "continue anyway" -- the read-only constraint applies to this shell exactly as it applies to every other feature Dana can view (XBR-29).
- Every guided step's own record creation (Client, Project, Branding Profile, Payment Account Connection, Proposal) is owned entirely by that step's own feature; this shell's chrome (progress indicator, skip affordance, continue-anyway pattern) is described identically wherever those screens are reached through onboarding, per the Brief's Shared Context.

## Edge Cases

- **Nadia closes the browser mid-step and returns later** -- Onboarding-progress state read on the next open resumes exactly where she left off; no step is re-shown as incomplete if it already completed before she left.
- **Nadia navigates directly to a step's URL out of order (e.g., proposal drafting before adding a client)** -- The destination screen (FEAT-02.SPEC-001) itself requires an existing project; with none yet, it redirects back to this shell's current step rather than opening in an invalid state. This is the destination screen's own precondition, not an onboarding gate; free navigation elsewhere is unaffected (R-08).
- **Nadia sees the "How did you hear" question, closes the browser, and returns later** -- The question is unresolved, so the Welcome state is shown again; the referring-portal reference persisted at sign-up is unaffected and is handed off when she answers or skips.
- **Nadia skips branding, then returns to it later from Settings (FEAT-21)** -- Setting branding later has no effect on this shell; onboarding's own progress indicator already marked that step skipped and does not revisit it.
- **A guided step's own action fails (e.g., a logo upload inside FEAT-19's branding step)** -- Per FEAT-20.SPEC-005's failure-tolerance rule, progression continues to the next step regardless; the failed step remains completable later from Settings, with no lost progress.
- **Dana opens a support session while Nadia is mid-step in her own session** -- Both views read the same onboarding-progress state; Dana's view has no controls to conflict with Nadia's, so there is no concurrent-edit conflict to resolve (Freelancer Account's Contention note states no contention arises from Dana's read-only access).
- **Connectivity is lost while a guided step that creates a record (client, project) is open** -- That step's own screen (FEAT-01.SPEC-001, FEAT-01.SPEC-002) shows its own offline message and preserves typed-but-unsaved input; this shell itself shows no error, since it creates nothing directly.
- **All mandatory steps complete out of the shell's suggested order (e.g., Nadia opens the proposal draft screen directly after adding a client, skipping past the shell's own "Draft the first proposal" button)** -- FEAT-20.SPEC-003 detects completion from the resulting records regardless of navigation path, and the completion is already recorded by the time she next opens this location (FEAT-20.SPEC-003 triggers from the record saves themselves, and re-checks on every shell open); this shell then shows the Ready state (if she has not yet acknowledged it).
- **A completion check cannot finish (including for the final step)** -- The shell shows the Completion-check notice with "Check again"; the check also re-runs on every shell open and on every later step completion or record save, so Nadia is never stranded without Ready.
- **Nadia opens the location after tapping "Go to your dashboard"** -- She is redirected to FEAT-12 with no message; the Ready state is not shown twice.
- **The welcome-email warning and a dismissed state** -- Once dismissed the warning is never shown again, even if a later delivery failure occurs for that same email; changing the sign-in email (FEAT-21.SPEC-005) also removes it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Navigation (inbound) | Successful account creation opens this shell on the Welcome state |
| FEAT-20.SPEC-003 (Onboarding Completion Detection) | Triggers (outbound) / References (inbound) | Every step completion, every shell open while In Progress, and every "Check again" tap re-checks exit criteria; this shell renders the Ready state or the Completion-check notice the automation reports |
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | Triggers (outbound) | Answering or skipping "How did you hear" triggers the capture hand-off |
| FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules) | References (inbound) | Governs mandatory/optional status, skip behavior, and failure tolerance applied by this shell |
| FEAT-01.SPEC-001 (Add Client) | Navigation (outbound) | "Add first client and project" step entry point |
| FEAT-19.SPEC-001 (Branding Settings) | Navigation (outbound) | "Set branding" step entry point |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Navigation (outbound) | "Connect payments" step entry point |
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Navigation (outbound) | "Draft the first proposal" step entry point |
| FEAT-12 (Freelancer Financial Dashboard) | Navigation (outbound) | The Ready state's "Go to your dashboard" button, and the redirect on any open after acknowledgement, lead to the dashboard; FEAT-12 is otherwise unaffected by onboarding |
| FEAT-20.SPEC-006 (Welcome Email) | References (inbound) / Navigation (inbound) | Supplies the delivery-failure outcome that drives the welcome-email warning defined here; its "Continue setup" link opens this location |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | References (inbound) | The project-level delivery warning this account-level variant stands in for while no project exists |
| FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | References (inbound) | Source of the welcome email's final delivery outcome |
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Navigation (outbound) | The warning's "Go to Settings" button leads here so Nadia can correct her sign-in email |
| FEAT-05 (Client Portal Access) | Navigation (outbound) | Owen and Priya's "Back to your portal" destination from the unavailable-page experience |
| FEAT-31 (Operator Support Access) | Navigation (inbound) / (outbound) | Dana's read-only session view of this shell; Dana's "Go to support access" destination when she reaches it outside a session |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|-------------------|----------------------|
| onboarding_step_completed | step name, time since previous step, step index | A guided step's own screen reports completion | supports success-metrics.md: "First-Session Activation" |
| onboarding_step_skipped | step name (branding or payments) | Nadia taps "Skip for now" on a skippable step | supports success-metrics.md: "First-Session Activation" (a skip is a completed, if lighter, path through the same session-activation flow) |
| how_did_you_hear_answered | answered vs. skipped | Nadia answers or skips the "How did you hear" question | supports success-metrics.md: "Growth Through Referral" |
| onboarding_step_failed_continued | step name, failure reason | A step's own action fails and this shell allows progression regardless | N/A -- no Stage 2 metric measures individual step failures; retained so failure-tolerance is observable rather than invisible |

## Acceptance Criteria

**FEAT-20.SPEC-002-AC-01:** Given Nadia has just created her account, when the guided sequence first opens, then she sees the Welcome message and the "How did you hear about us?" question.

**FEAT-20.SPEC-002-AC-02:** Given Nadia is on the Welcome state, when she types an answer and taps Continue, then FEAT-20.SPEC-004 is triggered with her answer (onboarding_how_did_you_hear_resolved is set true by FEAT-20.SPEC-004, not by this screen) and she advances to the "Add first client and project" step.

**FEAT-20.SPEC-002-AC-03:** Given Nadia is on the Welcome state, when she taps Skip instead of answering, then FEAT-20.SPEC-004 is triggered with "unknown" as the answer and she advances to the next step.

**FEAT-20.SPEC-002-AC-04:** Given Nadia is on the "Add first client and project" step, when she taps the button, then she is taken to FEAT-01.SPEC-001 (Add Client), and no "Skip for now" link is shown for this mandatory step.

**FEAT-20.SPEC-002-AC-05:** Given Nadia completes adding a client and project and returns to this shell, when the return is processed, then `onboarding_step_completed` is emitted, FEAT-20.SPEC-003 re-checks exit criteria, and she advances to the "Set your branding" step.

**FEAT-20.SPEC-002-AC-06:** Given Nadia is on the "Set your branding" step, when she taps "Skip for now", then `onboarding_step_skipped` is emitted, the confirmation "You can set this up anytime from Settings." appears, and she advances to "Connect payments" with no penalty.

**FEAT-20.SPEC-002-AC-07:** Given Nadia is on the "Connect payments" step, when she taps "Skip for now", then `onboarding_step_skipped` is emitted and she advances to "Draft your first proposal".

**FEAT-20.SPEC-002-AC-08:** Given Nadia has added a first client and project, when she reaches the "Draft the first proposal" step and taps its button, then she is taken to FEAT-02.SPEC-001 for the new project.

**FEAT-20.SPEC-002-AC-09:** Given Nadia has a first client, project, and drafted proposal, when FEAT-20.SPEC-003 reports the exit criteria met, then this shell shows the Ready state with a "Go to your dashboard" button, does not route automatically, and renders no step controls.

**FEAT-20.SPEC-002-AC-10:** Given Nadia is on the Ready state, when she taps "Go to your dashboard", then onboarding_ready_acknowledged becomes true and she is routed to FEAT-12 (Freelancer Financial Dashboard).

**FEAT-20.SPEC-002-AC-11:** Given Nadia's branding logo upload fails inside the "Set branding" step, when she returns to this shell, then progression continues to the next step regardless, the step is marked "Finish later from Settings," and `onboarding_step_failed_continued` is emitted, per FEAT-20.SPEC-005 Rule R-04 for optional steps.

**FEAT-20.SPEC-002-AC-12:** Given Dana opens a read-only support session on Nadia's account, when she views this shell, then she sees the identical progress indicator and current step with no step-entry buttons, "Skip for now," or "continue anyway" controls rendered.

**FEAT-20.SPEC-002-AC-13:** Given Owen (Client Primary Contact) or Priya (Client Reviewer Contact) in a portal session attempts to reach this location, when the attempt is made, then he or she sees the page "This page isn't available" with "That page isn't part of your portal." and a "Back to your portal" button leading to their portal home, and no onboarding data is loaded.

**FEAT-20.SPEC-002-AC-14:** Given Nadia closes the browser mid-step and signs back in later, when this shell reopens, then it resumes on the exact step reflected by her onboarding-progress state.

**FEAT-20.SPEC-002-AC-15:** Given Nadia loses connectivity while this shell is open, when the loss occurs, then the banner "This step needs a connection." appears in the body panel; if she then opens a step that creates a record, that step's own screen shows its own offline behavior.

**FEAT-20.SPEC-002-AC-16:** Given Nadia navigates directly to the proposal-draft screen's URL before adding any client, when the screen loads, then it redirects her back to this shell's current step rather than opening in an invalid state.

**FEAT-20.SPEC-002-AC-17:** Given Nadia completes her first client, project, and proposal draft by navigating directly to each feature's screen rather than tapping this shell's own buttons, when she next opens this shell, then FEAT-20.SPEC-003 has already recorded completion (triggered by the record saves) and the Ready state is shown, provided she has not yet tapped "Go to your dashboard".

**FEAT-20.SPEC-002-AC-18:** Given an unauthenticated visitor attempts to reach this shell's URL, when the attempt is made, then she is redirected to the sign-in screen.

**FEAT-20.SPEC-002-AC-19:** Given the read of onboarding-progress state fails when this shell opens, when the failure occurs, then the banner "We couldn't load your setup progress." appears with a Retry button and no step panel is shown.

**FEAT-20.SPEC-002-AC-20:** Given Nadia saw the "How did you hear" question, closed the browser without answering, and signs in again, when the shell opens, then onboarding_how_did_you_hear_resolved is still false and the Welcome state is shown again; after she answers or skips, later opens never show it.

**FEAT-20.SPEC-002-AC-21:** Given onboarding is Complete and Nadia has already tapped "Go to your dashboard", when she opens this location by sign-in, bookmark, or the welcome email's "Continue setup" link, then she is redirected to FEAT-12 with no message and the Ready state is not shown.

**FEAT-20.SPEC-002-AC-22:** Given Nadia's project creation fails inside the "Add first client and project" step, when she returns to this shell, then the same step's panel shows "Not finished" with a "Try again" button and the message "That didn't finish saving. Try again when you're ready.", with no "Skip for now" or "continue anyway" control.

**FEAT-20.SPEC-002-AC-23:** Given Nadia saves her proposal draft (the final mandatory step) and FEAT-20.SPEC-003 cannot finish its check, when she next sees this shell, then the Completion-check notice "We couldn't confirm your setup just now." with a "Check again" button is shown, and tapping "Check again" (or the next shell open) re-runs the check and shows the Ready state once it succeeds.

**FEAT-20.SPEC-002-AC-24:** Given the welcome email's retries were exhausted without delivery and Nadia has not dismissed the warning, when she opens this shell in the In Progress, Welcome, or Ready state, then the warning "We couldn't deliver your welcome email to {sign_in_email}. If that address is wrong, you can correct it in Settings." is shown with "Go to Settings" and "Dismiss".

**FEAT-20.SPEC-002-AC-25:** Given the welcome-email warning is shown, when Nadia taps "Go to Settings" she is taken to FEAT-21.SPEC-005, and when she taps "Dismiss" onboarding_welcome_warning_dismissed becomes true and the warning is never shown again.

**FEAT-20.SPEC-002-AC-26:** Given Dana has no open support session on Nadia's account, when she reaches this location, then she sees "This page isn't available" with "Open a support session on a freelancer's account to view their setup progress." and a "Go to support access" button, and no freelancer data is loaded.

**FEAT-20.SPEC-002-AC-27:** Given onboarding is In Progress, when Nadia opens the dashboard or Settings directly, then it opens normally with no gate or redirect back to this shell, and her next sign-in lands on this shell at her current step.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Interactions | 20 | 20 |
| States | 9 (loading, load error, welcome, in progress, completion check pending, ready, redirected, read-only, offline) | 9 |
| Business Rules | 8 | 8 |
| Edge Cases | 11 | 11 |



# Automation Spec: Onboarding Completion Detection

## Overview

**Name:** Onboarding Completion Detection
**ID:** FEAT-20.SPEC-003
**Type:** Automation
**Purpose:** Detects when a first client, project, and drafted proposal all exist for the Freelancer Account, marks onboarding complete, and signals FEAT-20.SPEC-002 to render the Ready state. This automation only signals; FEAT-20.SPEC-002 owns the Ready state and the route into the dashboard.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- Re-checking onboarding's exit criteria (a first Client, a first Project under it, and a drafted Proposal for that project) each time a guided step reports completion, each time the onboarding shell opens while onboarding is In Progress, and each time Nadia taps "Check again"
- Reporting a check that could not finish, so the shell can offer a retry
- When completing with the "How did you hear" question unresolved, auto-resolving it as "unknown" and firing the FEAT-20.SPEC-004 hand-off
- Marking onboarding complete on the Freelancer Account's onboarding-progress state exactly once
- Emitting `onboarding_completed` and signaling FEAT-20.SPEC-002 to render the Ready state
- Detecting completion regardless of whether the underlying records were created through the guided sequence's own buttons or by navigating directly to each feature's screen

**Non-Goals:**
- Defining which steps are mandatory for exit -- owned by FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules); this automation only evaluates the exit criteria that spec defines, it does not define them
- Creating the Client, Project, or Proposal records themselves -- owned by FEAT-01 and FEAT-02 respectively; this automation only reads their existence and state
- Routing, navigation, and the Ready state itself -- the Ready state and the "Go to your dashboard" route belong to FEAT-20.SPEC-002, and the dashboard's content to FEAT-12 (Freelancer Financial Dashboard); this automation never navigates and only reports an outcome
- Re-evaluating completion after onboarding has already been marked complete once -- excluded per the product's Entity-Lifecycle Coverage Matrix, which treats onboarding as a one-time, per-account setup sequence with no independent re-run

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A guided step reports completion | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires every time Nadia returns from a step's own screen having completed it, and onboarding is not already marked complete | Freelancer Account reference, the step just completed |
| The onboarding shell opens while onboarding is In Progress | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires on every open of the shell after its progress read succeeds, while onboarding is not already marked complete (covers a completion check that failed earlier, or completion that happened in another tab) | Freelancer Account reference |
| Nadia taps "Check again" on the completion-check notice | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires only when a previous check reported a read failure and onboarding is not already marked complete | Freelancer Account reference |
| A first client, project, or proposal draft is created outside the guided sequence's own navigation | FEAT-01.SPEC-002 (Create Project), FEAT-02.SPEC-001 (Proposal Draft Editor) | Fires when Nadia reaches a mandatory milestone (project creation, proposal draft save) by navigating directly rather than through FEAT-20.SPEC-002's buttons, and onboarding is not already marked complete | Freelancer Account reference, the record just created |

## Processing Logic

1. Confirm the Freelancer Account exists and onboarding is not already marked complete on its onboarding-progress state (per the Non-Goals above, a completed account never re-enters this check).
2. Read whether at least one Client exists under the Freelancer Account.
3. Read whether at least one Project exists under that Client (or any Client, if more than one now exists).
4. Read whether a Proposal in Draft or later status exists for that Project.
5. Evaluate the exit criteria defined by FEAT-20.SPEC-005: all three of Client, Project, and drafted Proposal must exist. Branding, payment connection, and full billing setup are never part of this evaluation, regardless of their skip state.
6. If all three exist, mark the Freelancer Account's onboarding-progress state complete, record the completion timestamp, and emit `onboarding_completed`. If onboarding_how_did_you_hear_resolved is still false, trigger FEAT-20.SPEC-004 with "unknown" (FEAT-20.SPEC-005 Rule R-09); this spec never writes onboarding_how_did_you_hear_resolved, FEAT-20.SPEC-004 sets it after its hand-off attempt.
7. If any of the three is missing, take no action -- onboarding-progress state remains unchanged, and FEAT-20.SPEC-002 continues to show the current in-progress step.
7a. If the read in steps 2-4 cannot complete, take no data action and treat the outcome as "check could not finish".
8. Signal FEAT-20.SPEC-002 with the outcome (complete, not yet, or check could not finish) so the shell can render the Ready state, remain on its current step, or show the completion-check notice with "Check again".

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|-----------------|--------------------|
| Exit criteria not yet met | One or more of Client, Project, drafted Proposal is missing | None | None directly -- FEAT-20.SPEC-002 continues showing the current step | FEAT-20.SPEC-002 |
| Onboarding marked complete | Client, Project, and a drafted Proposal all exist, and onboarding was not already complete | Freelancer Account's onboarding-progress state set to complete with a completion timestamp; FEAT-20.SPEC-004 is triggered (and it, not this spec, sets onboarding_how_did_you_hear_resolved) if the question was still unresolved | FEAT-20.SPEC-002 renders the Ready state (Nadia leaves it by tapping "Go to your dashboard"; no automatic routing) | FEAT-20.SPEC-002, FEAT-20.SPEC-004 (only when the question was unresolved) |
| Already complete (re-trigger ignored) | This automation fires again after onboarding is already marked complete | None | None -- silent no-action, since re-checking a settled outcome has no visible effect | -- |
| Read failure | The check for Client, Project, or Proposal existence cannot complete (e.g., a transient failure reading a shared entity) | None | FEAT-20.SPEC-002 shows the notice "We couldn't confirm your setup just now." with a "Check again" button. The check re-runs when Nadia taps "Check again", on every later shell open, and on every later step completion or record save -- so a failure on the final step never strands her | FEAT-20.SPEC-002 |

## Data Model

**Reads:** Freelancer Account -- to confirm the account exists and to read its onboarding-progress state. Client, Project (owned and created by FEAT-01) -- to confirm a first client and project exist. Proposal (owned and created by FEAT-02) -- to confirm a drafted proposal exists for the new project.
**Creates:** None.
**Updates:** Freelancer Account -- its onboarding-progress state only: onboarding_status set to complete with a completion timestamp. onboarding_how_did_you_hear_resolved is read but never written here (FEAT-20.SPEC-004 is its only writer). No other Freelancer Account field is touched.
**Deletes:** None.

## Business Rules

- The exit criteria are exactly the three named in FEAT-20.SPEC-005: a first Client, a first Project, and a drafted Proposal -- branding, payment connection, and full billing setup are optional and never gate completion (product-features.md, Validation & Limits).
- Completion is detected regardless of navigation path: whether Nadia used FEAT-20.SPEC-002's own step buttons or navigated directly into FEAT-01 or FEAT-02's screens, the same three records satisfy the same check.
- A check that cannot finish is never silent-and-final: it is reported to FEAT-20.SPEC-002 and re-runs on the next shell open, the next step completion or record save, or a "Check again" tap.
- Completion is recorded exactly once; once marked complete, this automation takes no further action on subsequent triggers for the same account.
- A Proposal in any status of Draft or later (Sent, Voided, Accepted) satisfies the "drafted proposal" criterion -- a proposal that has since been sent or accepted still means a draft was reached.

## Edge Cases

- **Nadia creates a project and drafts a proposal for it, then that project is archived before onboarding's check runs again** -- Archiving a project (FEAT-01) does not remove its historical existence; the exit criteria evaluate whether the records were ever created, not whether they remain active, so completion is still detected.
- **Nadia adds a client but no project yet** -- Exit criteria are not met; onboarding remains in progress and no completion is recorded.
- **The Proposal is later voided and re-sent (FEAT-02.SPEC-006)** -- The original draft already satisfied the exit criterion before the void; a Proposal's later lifecycle changes never retroactively un-complete onboarding, since completion is a one-time, never-reversed transition.
- **Concurrent trigger firing (Nadia completes the client/project step and, in a second open tab, drafts the proposal at effectively the same moment)** -- Both triggers independently read the current record state; whichever evaluation runs second sees the results of the first and correctly finds all three records present, marking completion once. The one-time completion rule (Business Rules) prevents a duplicate `onboarding_completed` emission even if both evaluations reach the completion condition before either writes.
- **Trigger fires while a previous run is in flight** -- A second evaluation for the same account is not started while the account's onboarding-progress state is mid-update from a prior evaluation; it re-reads the (by-then-updated) state instead, so it correctly finds onboarding already complete and takes no action.
- **The Freelancer Account is deleted (FEAT-24) between a step completing and this automation evaluating** -- The automation's read of the account fails; per the Read Failure outcome, no action is taken and there is no shell left to signal.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Triggered by (inbound) / Affects (outbound) | Every step completion, every shell open while In Progress, and every "Check again" tap triggers this check; the outcome tells the shell whether to advance, show Ready, or show the completion-check notice |
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | Triggers (outbound) | Completion with the "How did you hear" question unresolved auto-resolves it as "unknown" and fires the hand-off once |
| FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules) | References (inbound) | Defines which three records constitute the exit criteria this automation evaluates |
| FEAT-01.SPEC-002 (Create Project) | Triggered by (inbound) | A project created by navigating directly (outside the guided sequence's buttons) also re-triggers this check |
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Triggered by (inbound) | A proposal drafted by navigating directly also re-triggers this check |
| FEAT-12 (Freelancer Financial Dashboard) | References (outbound, indirect) | This automation does not route to or write to FEAT-12; the Ready state (FEAT-20.SPEC-002) it triggers is what offers Nadia the "Go to your dashboard" button |

## Analytics and Success Signals

- **onboarding_completed** (time from account creation to completion, whether branding was set, whether payments were connected) -- supports success-metrics.md: "First-Session Activation"
- **onboarding_exit_criteria_check_no_action** (which of the three criteria is still missing) -- N/A -- no Stage 2 metric measures individual in-progress checks; retained only as an internal no-action outcome, not a candidate for a dedicated signal beyond the completion event above.

## Acceptance Criteria

**FEAT-20.SPEC-003-AC-01:** Given Nadia has added a first client and project but has not yet drafted a proposal, when this automation evaluates the exit criteria, then onboarding is not marked complete and FEAT-20.SPEC-002 remains on its current step.

**FEAT-20.SPEC-003-AC-02:** Given Nadia has a first client, project, and drafted proposal, when this automation evaluates the exit criteria after the proposal draft is saved, then onboarding is marked complete, `onboarding_completed` is emitted, and FEAT-20.SPEC-002 shows the Ready state.

**FEAT-20.SPEC-003-AC-03:** Given onboarding is already marked complete, when this automation is triggered again (e.g., by an unrelated later action), then no action is taken and no duplicate `onboarding_completed` is emitted.

**FEAT-20.SPEC-003-AC-04:** Given Nadia skipped branding and payment connection entirely, when she completes the client, project, and proposal draft, then onboarding is marked complete regardless of the skipped steps.

**FEAT-20.SPEC-003-AC-05:** Given Nadia creates her first project by navigating directly to FEAT-01.SPEC-002 rather than through FEAT-20.SPEC-002's own button, when the project is created, then this automation still re-evaluates the exit criteria from that trigger.

**FEAT-20.SPEC-003-AC-06:** Given Nadia's drafted proposal is later voided and re-sent, when this automation is asked to re-evaluate at any later point, then onboarding remains complete -- the earlier draft already satisfied the criterion and completion is never reversed.

**FEAT-20.SPEC-003-AC-07:** Given Nadia completes her client/project step in one browser tab and drafts her proposal in another at effectively the same moment, when both triggers fire, then onboarding is marked complete exactly once, with no duplicate `onboarding_completed` event.

**FEAT-20.SPEC-003-AC-08:** Given a prior evaluation for Nadia's account is still updating onboarding-progress state, when a new trigger fires for the same account before that update finishes, then the new evaluation reads the just-updated state and correctly takes no further action.

**FEAT-20.SPEC-003-AC-09:** Given Nadia's Freelancer Account is deleted between a step completing and this automation's evaluation, when the automation attempts to read the account, then it takes no action and nothing is signaled to any shell.

**FEAT-20.SPEC-003-AC-10:** Given Nadia has a first client and project, when a Proposal for that project exists in any status of Draft or later, then the "drafted proposal" exit criterion is satisfied.

**FEAT-20.SPEC-003-AC-11:** Given the check for Nadia's final mandatory step could not read the proposal, when the failure occurs, then FEAT-20.SPEC-002 shows "We couldn't confirm your setup just now." with "Check again", and tapping it re-runs this automation and, once it succeeds, onboarding is marked complete and the Ready state is shown.

**FEAT-20.SPEC-003-AC-12:** Given a previous check failed and Nadia closes and reopens the shell, when the shell opens while onboarding is In Progress, then this automation re-runs on open and marks onboarding complete if all three records exist.

**FEAT-20.SPEC-003-AC-13:** Given onboarding completes, when the outcome is signalled, then FEAT-20.SPEC-002 renders the Ready state and this automation performs no navigation of its own.

**FEAT-20.SPEC-003-AC-14:** Given onboarding completes while onboarding_how_did_you_hear_resolved is false, when completion is recorded, then FEAT-20.SPEC-004 is triggered once with "unknown" and this spec does not write the field (FEAT-20.SPEC-004 sets it true after its hand-off attempt).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Referral Attribution Capture Hand-off

## Overview

**Name:** Referral Attribution Capture Hand-off
**ID:** FEAT-20.SPEC-004
**Type:** Automation
**Purpose:** Captures Nadia's optional "How did you hear about us?" answer and the referring-portal reference, then hands both to Portal Referral Attribution (FEAT-33) for recording.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- Capturing the self-reported answer (or "unknown" when skipped) at the moment Nadia answers or skips the question in FEAT-20.SPEC-002
- Reading the referring-portal reference that FEAT-20.SPEC-001 persisted on the Freelancer Account (`onboarding_referring_portal_ref`) at sign-up, or recording it as absent when the visitor arrived without one
- Handing both values off to FEAT-33 exactly once per Freelancer Account
- Tolerating the hand-off's own failure without blocking or reversing onboarding progress

**Non-Goals:**
- Creating or owning the Referral Attribution record itself -- per the dependency map, "Created by FEAT-33 at sign-up (answer captured in FEAT-20)"; this automation only captures and hands off the two values, FEAT-33 persists them
- Displaying the "How did you hear" question -- owned by FEAT-20.SPEC-002; this automation begins where that screen's answer/skip interaction ends
- Measuring or reporting the referral growth loop -- owned by FEAT-33's own aggregate measurement (success-metrics.md, "Growth Through Referral"); this automation only supplies the raw values
- Re-asking or re-capturing the answer after it is resolved -- the question is resolved exactly once, per FEAT-20.SPEC-002's Business Rules and FEAT-20.SPEC-005 R-06; this automation never fires a second time for the same account

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia answers the "How did you hear" question | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires once, when Nadia taps Continue after entering an answer | Freelancer Account reference, the entered answer text; the referring-portal reference (if any) is read from the account's onboarding_referring_portal_ref |
| Nadia skips the "How did you hear" question | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires once, when Nadia taps Skip without entering an answer | Freelancer Account reference, no answer text (treated as "unknown"); the referring-portal reference (if any) is read from the account's onboarding_referring_portal_ref |
| Onboarding completes with the question unresolved | FEAT-20.SPEC-003 (Onboarding Completion Detection) | Fires once, when completion is recorded while onboarding_how_did_you_hear_resolved is false (FEAT-20.SPEC-005 R-09) | Freelancer Account reference, no answer text (treated as "unknown"); the referring-portal reference (if any) read from the account |

## Processing Logic

1. Receive the Freelancer Account reference and the answer (text or absent) from the trigger, then read onboarding_referring_portal_ref (present or absent) from the account; if onboarding_how_did_you_hear_resolved is already true, stop -- the hand-off has already fired.
2. If no answer was entered (the Skip path), set the self-reported source value to "unknown."
3. If a referring-portal reference is absent (the visitor arrived without following a FEAT-33 mark), record it as absent rather than substituting any inferred value.
4. Hand off the self-reported source value and the referring-portal reference (or its absence) to Portal Referral Attribution (FEAT-33) for it to create the Referral Attribution record, identifying the hand-off by the Freelancer Account reference so FEAT-33 creates at most one Referral Attribution record per account.
5. Whether the hand-off in step 4 succeeded or failed, set onboarding_how_did_you_hear_resolved to true on the account with a compare-and-set (change false to true only if it is still false). This automation is the only writer of onboarding_how_did_you_hear_resolved after account creation: FEAT-20.SPEC-002 and FEAT-20.SPEC-003 only trigger this automation and never write the field. If the compare-and-set finds the field already true (another run resolved it first), do nothing further.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|-----------------|--------------------|
| Hand-off succeeds with an answer | Nadia entered an answer and the hand-off to FEAT-33 completes | FEAT-33 creates its Referral Attribution record with the self-reported source and referring-portal reference; this automation then sets onboarding_how_did_you_hear_resolved to true | None -- Nadia already saw her onboarding step advance in FEAT-20.SPEC-002; this hand-off is invisible to her | FEAT-33 (Referral Attribution record) |
| Hand-off succeeds with "unknown" | Nadia skipped the question and the hand-off to FEAT-33 completes | FEAT-33 creates its Referral Attribution record with self-reported source "unknown"; this automation then sets onboarding_how_did_you_hear_resolved to true | None | FEAT-33 (Referral Attribution record) |
| Hand-off fails | The hand-off to FEAT-33 cannot complete (e.g., a transient failure) | No Referral Attribution record is created at this time; onboarding_how_did_you_hear_resolved is still set to true (compare-and-set), so the question is resolved and never re-asked and the hand-off is not retried | None -- onboarding progression in FEAT-20.SPEC-002 is never blocked or delayed by this failure | FEAT-20.SPEC-002 (unaffected) |

## Data Model

**Reads:** Freelancer Account -- onboarding_referring_portal_ref (persisted at sign-up by FEAT-20.SPEC-001) and onboarding_how_did_you_hear_resolved, plus the answer text supplied by the trigger.
**Creates:** None directly -- the Referral Attribution record is created by FEAT-33, not by this automation.
**Updates:** Freelancer Account -- onboarding_how_did_you_hear_resolved set true by compare-and-set after the hand-off attempt (this automation is its only writer after account creation, whether the hand-off succeeded or failed); no other field.
**Deletes:** None.

## Business Rules

- This automation fires at most once per Freelancer Account, in lockstep with the question being resolved exactly once (answered, skipped, or auto-resolved at onboarding completion). Because the question is resolved only on answer or skip -- not on render -- a Nadia who saw it and left is asked again on return and the hand-off still fires when she finally resolves it.
- This automation is the single writer of onboarding_how_did_you_hear_resolved (after its default of false at account creation). It performs the hand-off first and then sets the field true by compare-and-set; the value after a failed hand-off is also true, because the question has been answered, skipped, or auto-resolved and is never re-asked. FEAT-20.SPEC-002 and FEAT-20.SPEC-003 only trigger this automation.
- A skipped question is captured as "unknown," never as an empty or missing value -- FEAT-33's Referral Attribution record always receives a defined self_reported_source value.
- The referring-portal reference, when present, is passed through unchanged from the value persisted on the account at sign-up (FEAT-20.SPEC-001) -- this automation never re-derives or re-validates it.
- This hand-off is non-blocking: onboarding's own progression (advancing to the next guided step) never waits on or is reversed by the hand-off's outcome (XBR-32: the referral mark's attribution is used only in aggregate, never a gate on the freelancer's own experience).

## Edge Cases

- **The hand-off to FEAT-33 fails** -- No retry is attempted from this automation; onboarding continues normally, and the self-reported source is simply never recorded for this account, and onboarding_how_did_you_hear_resolved is still set to true (accepted per this feature's non-blocking business rule -- a failed attribution never re-surfaces to Nadia, since she is never asked twice per FEAT-20.SPEC-002).
- **Nadia enters free text that is empty after trimming whitespace** -- Treated identically to an explicit Skip: the self-reported source is recorded as "unknown."
- **A referring-portal reference is present but that referring Freelancer Account is later deleted (FEAT-24)** -- The hand-off already completed at sign-up time with the reference as it stood then; FEAT-33 owns how a later deletion affects an already-recorded attribution, which is outside this automation's scope.
- **Nadia sees the question, closes the browser without answering, and returns** -- Nothing has fired and onboarding_how_did_you_hear_resolved is false; the persisted reference is intact; the hand-off fires when she answers or skips on her return.
- **Onboarding completes before Nadia ever answers or skips** -- The third trigger fires once with "unknown" and the persisted reference, so the reference is not lost; the question is not shown afterward.
- **Concurrent trigger firing (e.g., Nadia answers in one tab while a second tab skips, or completion lands at the same moment)** -- Each run first checks onboarding_how_did_you_hear_resolved and stops if it is already true. If two runs both pass that check, both hand off, but FEAT-33 treats the Freelancer Account reference as the identity of the hand-off and creates only one Referral Attribution record; only the run whose compare-and-set changes the field from false to true completes, the other does nothing further. The hand-off therefore results in exactly one record.
- **Trigger fires while a previous run is in flight** -- A second run for the same account re-reads onboarding_how_did_you_hear_resolved before handing off; once the earlier run has set it, the second run stops without a duplicate hand-off, and if it read the field before the earlier run set it, the same account-identified hand-off and compare-and-set rules above prevent a second record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Triggered by (inbound) | Answering or skipping the "How did you hear" question fires this hand-off; SPEC-002 only triggers it and never writes onboarding_how_did_you_hear_resolved |
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | References (inbound) | Persists the referring-portal reference carried forward from a FEAT-33 entry on the account at creation, where this automation reads it |
| FEAT-20.SPEC-003 (Onboarding Completion Detection) | Triggered by (inbound) | Completion with the question unresolved fires the hand-off once with "unknown"; SPEC-003 only triggers it and never writes onboarding_how_did_you_hear_resolved |
| FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules) | References (inbound) | Defines onboarding_how_did_you_hear_resolved and onboarding_referring_portal_ref, R-06 and R-09 |
| FEAT-33 (Portal Referral Attribution) | Affects (outbound) | Hands off the self-reported source and referring-portal reference for Referral Attribution creation |

## Analytics and Success Signals

- **how_did_you_hear_answered** (answered vs. skipped) -- supports success-metrics.md: "Growth Through Referral" (this event is also emitted by FEAT-20.SPEC-002 at the interaction level; this automation's own emission confirms the value actually reached the hand-off, distinguishing an answered-but-failed-hand-off case from a fully recorded one)
- **referral_attribution_handoff_failed** (had a referring-portal reference: yes/no) -- N/A -- no Stage 2 metric measures hand-off failures directly; retained so a silently dropped attribution is observable rather than invisible

## Acceptance Criteria

**FEAT-20.SPEC-004-AC-01:** Given Nadia types "A friend recommended it" and taps Continue on the "How did you hear" question, when this automation fires, then it hands off that answer to FEAT-33 for the Referral Attribution record.

**FEAT-20.SPEC-004-AC-02:** Given Nadia taps Skip without entering an answer, when this automation fires, then it hands off "unknown" as the self-reported source to FEAT-33.

**FEAT-20.SPEC-004-AC-03:** Given Nadia arrived by following a "Made with Clientroom" mark, when this automation fires (even in a later session than sign-up), then the referring-portal reference read from her account's onboarding_referring_portal_ref is included unchanged in the hand-off to FEAT-33.

**FEAT-20.SPEC-004-AC-04:** Given Nadia arrived without following any referral mark, when this automation fires, then the hand-off records the referring-portal reference as absent rather than substituting any inferred value.

**FEAT-20.SPEC-004-AC-05:** Given the hand-off to FEAT-33 fails, when the failure occurs, then Nadia's onboarding progression in FEAT-20.SPEC-002 continues unaffected and she is never re-asked the question.

**FEAT-20.SPEC-004-AC-06:** Given Nadia types only whitespace into the answer field before tapping Continue, when this automation fires, then the self-reported source is recorded as "unknown," identical to an explicit Skip.

**FEAT-20.SPEC-004-AC-07:** Given onboarding has already fired this hand-off once for Nadia's account, when any later action occurs, then this automation never fires a second time for that account.

**FEAT-20.SPEC-004-AC-08:** Given the hand-off succeeds, when it completes, then Nadia sees no confirmation of her own -- the hand-off is invisible, since her onboarding step already advanced in FEAT-20.SPEC-002.

**FEAT-20.SPEC-004-AC-09:** Given the hand-off fails for an account with a referring-portal reference present, when the failure occurs, then `referral_attribution_handoff_failed` is emitted noting a referring-portal reference was present, so the dropped attribution is observable.

**FEAT-20.SPEC-004-AC-10:** Given Nadia saw the "How did you hear" question, closed the browser without answering, and signed in again the next day, when she then answers, then this automation fires once and includes the referring-portal reference persisted at sign-up.

**FEAT-20.SPEC-004-AC-11:** Given onboarding completes while the question was never answered or skipped, when completion is recorded, then this automation fires once with "unknown" and the persisted reference.

**FEAT-20.SPEC-004-AC-12:** Given Nadia answers in one tab and skips in another at effectively the same moment, when both run, then FEAT-33 ends with exactly one Referral Attribution record for her account, only one run's compare-and-set changes onboarding_how_did_you_hear_resolved from false to true, and the other run does nothing further.

**FEAT-20.SPEC-004-AC-13:** Given Nadia answers or skips the question, when FEAT-20.SPEC-002 triggers this automation, then onboarding_how_did_you_hear_resolved is still false when the trigger arrives and becomes true only after this automation's hand-off attempt finishes, written by this automation and not by FEAT-20.SPEC-002.

**FEAT-20.SPEC-004-AC-14:** Given the hand-off to FEAT-33 fails, when this automation finishes, then onboarding_how_did_you_hear_resolved is true, the question is never shown again, and no retry of the hand-off occurs.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Onboarding Step Sequencing & Exit-Criteria Rules

## Overview

**Name:** Onboarding Step Sequencing & Exit-Criteria Rules
**ID:** FEAT-20.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs which onboarding steps are mandatory versus optional, the exit criteria for leaving onboarding, skip/resume behavior, and failure tolerance -- the single source of truth every other spec in this feature reads rather than duplicates.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup
**Governed Entity:** Freelancer Account (its onboarding-progress state)

## Scope and Non-Goals

**In Scope:**
- Which of the five guided steps ("How did you hear," Add first client and project, Set branding, Connect payments, Draft the first proposal) are mandatory versus optional
- The exit criteria that determine when onboarding is complete
- Skip behavior for optional steps, and the no-penalty resume path from Settings (FEAT-21)
- Failure-tolerance behavior when a step's own action fails
- Authorization for who may act on onboarding-progress state, and what an unauthorized person experiences
- Where the referring-portal reference and the "how did you hear" lifecycle live on the account, and what happens when Nadia leaves and returns
- The post-sign-in landing rule and the explicit absence of a navigation gate while onboarding is In Progress

**Non-Goals:**
- Rendering the steps, progress indicator, or any UI chrome -- owned by FEAT-20.SPEC-002 (Onboarding Guided Sequence); this spec defines the rules that screen applies, not its layout
- Evaluating whether the exit criteria are currently met -- owned by FEAT-20.SPEC-003 (Onboarding Completion Detection); this spec defines what the criteria are, that automation checks them
- Validating the content of each step's own record (a Client's fields, a Proposal's fields, and so on) -- owned by each destination feature's own Logic/Rule spec (e.g., FEAT-01's client validation); this spec governs only onboarding's own sequencing and exit state, not the data captured inside each step
- A configurable workflow or step builder that would let the rules here vary per account -- excluded per scope-boundaries.md SC-11: the product ships one fixed, sensible sequence for every freelancer

## Governed Entity

**Entity:** Freelancer Account (onboarding-progress state)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| onboarding_current_step | enum | Which of the five guided steps is currently active, or "complete" |
| onboarding_completed_steps | derived (set of enum) | Which steps have been completed |
| onboarding_skipped_steps | derived (set of enum) | Which optional steps have been explicitly skipped |
| onboarding_failed_incomplete_steps | derived (set of enum) | Which steps were attempted, failed their own action, and were continued past (still completable later from Settings) |
| onboarding_how_did_you_hear_resolved | boolean | Whether the "How did you hear" question has been answered or skipped (set on answer or skip, never on render), or auto-resolved as "unknown" because onboarding completed first |
| onboarding_referring_portal_ref | reference (nullable) | The referring Freelancer Account reference from a FEAT-33 entry, or absent; written once by FEAT-20.SPEC-001 at account creation and read by FEAT-20.SPEC-004 |
| onboarding_ready_acknowledged | boolean | Whether Nadia has tapped "Go to your dashboard" on the Ready state |
| onboarding_welcome_warning_dismissed | boolean | Whether Nadia has dismissed the welcome-email delivery warning shown in FEAT-20.SPEC-002 |
| onboarding_status | enum (In Progress / Complete) | Set to Complete by FEAT-20.SPEC-003 once the exit criteria are met |
| onboarding_completed_at | date-time | Timestamp written once, when onboarding_status transitions to Complete |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|----------------------|
| FEAT-20.SPEC-001 | Sign-Up & Account Creation | At account creation: initialises every default in the Defaults and Derivations table and writes onboarding_referring_portal_ref |
| FEAT-20.SPEC-002 | Onboarding Guided Sequence | On every step transition (entry, skip, continue-past-failure); authorization on screen entry; the post-sign-in landing rule (R-08); triggering FEAT-20.SPEC-004 on answer or skip (never writes onboarding_how_did_you_hear_resolved); the Ready acknowledgement |
| FEAT-20.SPEC-004 | Referral Attribution Capture Hand-off | Reads onboarding_referring_portal_ref; the only writer of onboarding_how_did_you_hear_resolved: guards the one-time hand-off by reading it, performs the hand-off, then sets it true by compare-and-set (true also when the hand-off fails) |
| FEAT-20.SPEC-003 | Onboarding Completion Detection | On every re-check of the exit criteria; triggers FEAT-20.SPEC-004 when completing with the question unresolved (never writes onboarding_how_did_you_hear_resolved) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| onboarding_current_step | Must be one of the five defined step identifiers, or "complete" | Always | On every write | No validation beyond data type -- this field is system-set, never user-entered, so there is no user-facing error to define | No |
| onboarding_completed_steps | May only gain entries, never lose one, once a step is marked complete | Always | On every write | No user-facing error -- this is a system-maintained set with no direct user input | No |
| onboarding_skipped_steps | May only contain steps defined as optional (Rule R-01 below) | Always | On every write | No user-facing error -- a mandatory step is never offered a skip control by FEAT-20.SPEC-002, so this condition cannot be violated through the product's own UI | No |
| onboarding_how_did_you_hear_resolved | Written only by FEAT-20.SPEC-004, by compare-and-set after its hand-off attempt (true whether the hand-off succeeded or failed); once set true, never reset to false; set true only when the question is answered, skipped, or auto-resolved at onboarding completion -- never merely because the question was rendered | Always | On every write | No validation beyond data type | No |
| onboarding_referring_portal_ref | Written exactly once, at account creation; never changed afterward | Always | On every write | No user-facing error -- system-set, never user-entered | No |
| onboarding_ready_acknowledged | May become true only while onboarding_status is Complete; once true, never reset | Always | On every write | No user-facing error -- system-set by FEAT-20.SPEC-002 | No |
| onboarding_welcome_warning_dismissed | Once set true, never reset to false | Always | On every write | No validation beyond data type | No |
| onboarding_status | May transition from In Progress to Complete exactly once; never reverses | Always | On every write | No user-facing error -- this transition is system-driven by FEAT-20.SPEC-003 | No |
| onboarding_completed_at | Set exactly once, at the same moment onboarding_status becomes Complete | onboarding_status transitions to Complete | On that transition only | No validation beyond data type | No |
| onboarding_failed_incomplete_steps | May only contain optional steps (Set branding, Connect payments), per Rule R-04; a failed mandatory step is never continued past and so never appears here | Always | On every write | No user-facing error -- FEAT-20.SPEC-002 offers "continue anyway" only on optional steps | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Exit criteria independence from skip/failure state | onboarding_status, onboarding_skipped_steps, onboarding_failed_incomplete_steps | onboarding_status may become Complete regardless of what onboarding_skipped_steps or onboarding_failed_incomplete_steps contain, as long as the three exit-criteria records exist (Rule R-04) | N/A -- no user-facing error; this is a system evaluation rule |
| One-time question | onboarding_how_did_you_hear_resolved, onboarding_current_step | Until Nadia answers or skips the question, onboarding_current_step stays "How did you hear" and, while onboarding_how_did_you_hear_resolved is false and onboarding_status is In Progress, the question is shown again on every return. On answer or skip, FEAT-20.SPEC-002 advances onboarding_current_step past "How did you hear" at once and triggers FEAT-20.SPEC-004 without waiting for its hand-off; onboarding_how_did_you_hear_resolved becomes true when that hand-off attempt finishes (or at Rule R-09 auto-resolution). The question is shown only while onboarding_current_step is "How did you hear", onboarding_how_did_you_hear_resolved is false and onboarding_status is In Progress; once the step has advanced or the field is true, the step is never re-entered | N/A -- FEAT-20.SPEC-002 renders the question until it is answered or skipped and never afterward |
| Ready acknowledgement | onboarding_status, onboarding_ready_acknowledged | onboarding_ready_acknowledged can be true only if onboarding_status is Complete; the Ready state renders only while status is Complete and acknowledged is false | N/A -- system evaluation rule |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View onboarding progress | Nadia (Freelancer) | Always, on her own account | -- |
| View onboarding progress | Dana (Support Operator) | Only inside a logged, time-limited support session on this specific account (FEAT-31) | Outside an open support session, the freelancer-side onboarding location is not served to her: she sees a plain page titled "This page isn't available" with the text "Open a support session on a freelancer's account to view their setup progress." and a "Go to support access" button leading to FEAT-31; no freelancer data is loaded |
| View onboarding progress | Owen (Client Primary Contact) | Never | The freelancer-side onboarding location is not served to a portal session: Owen sees a plain page titled "This page isn't available" with the text "That page isn't part of your portal." and a "Back to your portal" button leading to his portal home (FEAT-05); no onboarding data is loaded (Access field: "Nadia only") |
| View onboarding progress | Priya (Client Reviewer Contact) | Never | Identical to Owen: the same "This page isn't available" page with "That page isn't part of your portal." and "Back to your portal" leading to her portal home (FEAT-05) |
| Advance / skip / continue-past-failure a step | Nadia (Freelancer) | Always, on her own account, and only while onboarding_status is In Progress | Once onboarding_status is Complete, opening FEAT-20.SPEC-002 shows the Ready state if onboarding_ready_acknowledged is false; otherwise Nadia is redirected to the dashboard (FEAT-12) with no message. In neither case is any step control rendered -- there is nothing left to advance, skip, or continue past |
| Leave onboarding (open the dashboard or any other feature) while In Progress | Nadia (Freelancer) | Always -- no gate (Rule R-08) | N/A -- never denied; onboarding progress is preserved and she lands back in FEAT-20.SPEC-002 at her next sign-in |
| Advance / skip / continue-past-failure a step | Dana (Support Operator) | Never | No step-entry, skip, or continue controls render in Dana's read-only session (FEAT-20.SPEC-002); XBR-29 makes every operator surface read-only |
| Advance / skip / continue-past-failure a step | Owen, Priya (Client Contacts) | Never | Same page and message as View above ("This page isn't available" / "That page isn't part of your portal."); no step action is reachable |
| Mark onboarding complete (set onboarding_status to Complete) | System (FEAT-20.SPEC-003) only | Always, when the exit criteria are met | N/A -- no person performs this action directly; it is evaluated automatically |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| onboarding_current_step | "How did you hear" | On Freelancer Account creation -- applied by FEAT-20.SPEC-001, which initialises every default in this table in the same atomic step that creates the account (FEAT-20.SPEC-001 Business Rules) | No -- this is the fixed starting point for every account |
| onboarding_completed_steps | Empty set | On Freelancer Account creation | No (grows only through step completion) |
| onboarding_skipped_steps | Empty set | On Freelancer Account creation | No (grows only through an explicit Skip) |
| onboarding_failed_incomplete_steps | Empty set | On Freelancer Account creation | No (grows only through a step's own action failing) |
| onboarding_how_did_you_hear_resolved | false | On Freelancer Account creation | No (system-set to true when the question is answered or skipped, or auto-resolved at onboarding completion) |
| onboarding_referring_portal_ref | The FEAT-33 referring-portal reference carried into sign-up, or absent | On Freelancer Account creation | No (written once, never overridden) |
| onboarding_ready_acknowledged | false | On Freelancer Account creation | No (set true when Nadia taps "Go to your dashboard") |
| onboarding_welcome_warning_dismissed | false | On Freelancer Account creation | No (set true when Nadia dismisses the warning) |
| onboarding_status | In Progress | On Freelancer Account creation | No -- only FEAT-20.SPEC-003 transitions it to Complete |
| onboarding_completed_at | Unset | On Freelancer Account creation | No (set once, by FEAT-20.SPEC-003, never overridden) |

## Business Rules

- **R-01 (Mandatory vs. optional):** "Add first client and project" and "Draft the first proposal" are mandatory -- they cannot be skipped and are the two record-creating steps that constitute the exit criteria alongside the client itself. "Set branding," "Connect payments," and full billing setup are optional and skippable. The "How did you hear" question is optional to answer but is always shown until resolved (it is not itself skippable in the sense of being hidden -- Nadia either answers or explicitly skips it, per FEAT-20.SPEC-002's interaction).
- **R-02 (Exit criteria):** Onboarding's exit criteria are exactly three records: a first Client, a first Project under it, and a drafted Proposal for that project (product-features.md, Validation & Limits). No other step's completion or skip state affects this evaluation.
- **R-03 (Skip is permanent and penalty-free):** Skipping "Set branding" or "Connect payments" carries no penalty and no forced return -- Nadia may complete either later from Settings & Account Management (FEAT-21) at any time, with no re-entry into the onboarding shell required (per the Navigation Connections: "FEAT-21 settings -> FEAT-19 branding settings -- Nadia returns to a skipped branding step later").
- **R-04 (Failure tolerance):** "Never blocks progression" is defined separately for the two kinds of step.
  - *Optional steps (Set branding, Connect payments):* a failed step action (e.g., a logo upload failing inside FEAT-19's branding step) never blocks progression to the next step. FEAT-20.SPEC-002 offers "continue anyway"; the failed step is recorded in onboarding_failed_incomplete_steps and remains completable later from Settings, with no lost progress on any other step.
  - *Mandatory steps (Add first client and project, Draft the first proposal):* a failed step action (client or project creation failing, proposal drafting failing) means the required record does not exist, so the step cannot be continued past and is never added to onboarding_failed_incomplete_steps. It also never traps Nadia: the step stays current and marked "Not finished", FEAT-20.SPEC-002 offers "Try again" on the same launch panel (no "continue anyway"), all previously completed steps stay completed, and she may leave onboarding at any time (Rule R-08). Because the exit criteria (R-02) need those records, onboarding_status stays In Progress until a retry succeeds.
- **R-05 (One-time completion):** Once onboarding_status becomes Complete, it never reverts to In Progress, regardless of any later change to the underlying Client, Project, or Proposal records (e.g., the project being archived or the proposal being voided and re-sent).
- **R-06 (One-time question):** The "How did you hear" question is resolved exactly once per account. It is shown on every open of FEAT-20.SPEC-002 while onboarding_current_step is still "How did you hear", onboarding_how_did_you_hear_resolved is false and onboarding_status is In Progress -- including when Nadia saw it, closed the browser, and returned. On answer or skip, FEAT-20.SPEC-002 advances onboarding_current_step at once and triggers FEAT-20.SPEC-004 without waiting for the hand-off, so the question is not re-shown while that hand-off is still in flight. FEAT-20.SPEC-004 sets onboarding_how_did_you_hear_resolved true only when she answers, skips, or (Rule R-09) onboarding completes first; rendering the question sets nothing. FEAT-20.SPEC-004 is the single writer: it performs the hand-off, then sets the field true by compare-and-set, and the field is true even if the hand-off fails. FEAT-20.SPEC-002 and FEAT-20.SPEC-003 only trigger it. The hand-off fires exactly once, at that resolution.
- **R-07 (No configurable sequence):** The five-step sequence and its mandatory/optional split are the same for every account; scope-boundaries.md SC-11 excludes any per-account configuration of this order.
- **R-08 (Landing rule; no navigation gate):** While onboarding_status is In Progress, FEAT-20.SPEC-002 is Nadia's post-sign-in landing destination (also used by the sign-up redirect and the welcome-email CTA), so a returning Nadia is placed back on her current step. This is a landing rule, not a gate: no other feature (the dashboard, clients, projects, Settings) is blocked, redirected, or hidden while onboarding is In Progress, and leaving onboarding never changes onboarding-progress state. Once onboarding_status is Complete, the landing destination is the Ready state if onboarding_ready_acknowledged is false, otherwise the dashboard (FEAT-12).
- **R-09 (Completion with the question unresolved):** If onboarding_status becomes Complete while onboarding_how_did_you_hear_resolved is false (Nadia never answered or skipped), the question is not asked afterward: the account is auto-resolved as "unknown" (FEAT-20.SPEC-003 triggers FEAT-20.SPEC-004, which sets the field) and the FEAT-20.SPEC-004 hand-off fires once with the persisted onboarding_referring_portal_ref, so the reference is never lost.

## Edge Cases

- **Nadia skips branding, then completes it from Settings before finishing onboarding's mandatory steps** -- The skip already recorded in onboarding_skipped_steps is not reversed retroactively; onboarding-progress state simply reflects that branding was skipped during onboarding even though it is now also set, since R-03 grants no re-entry requirement.
- **A step is both skipped and later fails** -- Not possible: a skipped step's own action never runs during onboarding (R-03), so it cannot also appear in onboarding_failed_incomplete_steps from the same onboarding session; if Nadia later attempts it from Settings and it fails there, that failure is Settings' own concern (FEAT-21), not onboarding's.
- **Onboarding's exit criteria are met while a failed step is still outstanding** -- R-02 and R-04 combine cleanly: the failed step never gates the exit criteria (it was never one of the three), so completion proceeds normally with the failed step still listed as completable later.
- **Nadia attempts to skip "Add first client and project"** -- Not possible through the product's own UI: FEAT-20.SPEC-002 never renders a "Skip for now" control for a mandatory step (R-01), so this condition cannot arise through the defined interaction.
- **onboarding_status reaches Complete while the "How did you hear" question is unresolved (Nadia used the product freely and never answered or skipped)** -- Handled by R-09: the account is auto-resolved as "unknown", the hand-off fires once, and the question is never shown.
- **Nadia sees the question, closes the browser without answering, and signs in the next day** -- onboarding_how_did_you_hear_resolved is still false (render sets nothing), so FEAT-20.SPEC-002 shows the Welcome state again; the persisted onboarding_referring_portal_ref is intact and is handed off when she answers or skips.
- **A mandatory step's own action fails (e.g., project creation fails)** -- R-04: the step stays current as "Not finished" with "Try again"; nothing is added to onboarding_failed_incomplete_steps; exit criteria stay unmet.
- **Nadia opens the dashboard midway through onboarding** -- R-08: allowed with no gate or banner; her progress is preserved and the next sign-in lands her back on her current step.
- **Dana's support session is open at the exact moment Nadia completes the mandatory steps and onboarding transitions to Complete** -- Dana's read-only view (Authorization Rules) simply reflects the new Complete state on her next refresh; there is no conflict to resolve, since Dana never writes to onboarding-progress state.

## Acceptance Criteria

**FEAT-20.SPEC-005-AC-01:** Given Nadia is on the "Add first client and project" step, when FEAT-20.SPEC-002 renders it, then no "Skip for now" control is shown, per Rule R-01.

**FEAT-20.SPEC-005-AC-02:** Given Nadia is on the "Set branding" step, when FEAT-20.SPEC-002 renders it, then a "Skip for now" control is shown, per Rule R-01.

**FEAT-20.SPEC-005-AC-03:** Given Nadia has a first Client, Project, and drafted Proposal, when FEAT-20.SPEC-003 evaluates the exit criteria, then onboarding_status transitions to Complete regardless of branding or payment-connection state, per Rule R-02.

**FEAT-20.SPEC-005-AC-04:** Given Nadia skips "Connect payments," when she later opens Settings & Account Management (FEAT-21) and connects her payment account there, then no penalty or re-entry into onboarding is required, per Rule R-03.

**FEAT-20.SPEC-005-AC-05:** Given Nadia's logo upload fails inside the "Set branding" step, when she returns to FEAT-20.SPEC-002, then progression continues to the next step and the branding step is recorded in onboarding_failed_incomplete_steps as completable later from Settings, per Rule R-04.

**FEAT-20.SPEC-005-AC-06:** Given onboarding_status has already transitioned to Complete, when Nadia's project is later archived, then onboarding_status remains Complete, per Rule R-05.

**FEAT-20.SPEC-005-AC-07:** Given Nadia has answered or skipped the "How did you hear" question, when any later session opens FEAT-20.SPEC-002, then the question is never shown again, per Rule R-06.

**FEAT-20.SPEC-005-AC-08:** Given Nadia (Freelancer) is viewing her own onboarding progress, when she attempts any step action while onboarding_status is In Progress, then the action is allowed.

**FEAT-20.SPEC-005-AC-09:** Given onboarding_status has already transitioned to Complete, when Nadia opens FEAT-20.SPEC-002's URL, then she sees the Ready state if onboarding_ready_acknowledged is false, or is redirected to the dashboard (FEAT-12) with no message if it is true; in neither case is any step control rendered.

**FEAT-20.SPEC-005-AC-10:** Given Dana opens a logged support session on Nadia's account, when she views onboarding progress, then she can see it but no step-entry, skip, or continue-past-failure control is available to her, per the Authorization Rules.

**FEAT-20.SPEC-005-AC-11:** Given Dana's support session on this account is not open, when she attempts to reach the freelancer-side onboarding location, then she sees the page "This page isn't available" with "Open a support session on a freelancer's account to view their setup progress." and a "Go to support access" button, and no freelancer data is loaded.

**FEAT-20.SPEC-005-AC-12:** Given Owen (Client Primary Contact) or Priya (Client Reviewer Contact) in a portal session opens the freelancer-side onboarding location, then the page "This page isn't available" with "That page isn't part of your portal." and a "Back to your portal" button appears, and no onboarding data is loaded.

**FEAT-20.SPEC-005-AC-13:** Given a new Freelancer Account is created, when its onboarding-progress state is initialized, then onboarding_current_step defaults to "How did you hear," onboarding_status defaults to In Progress, and every set field defaults empty, with FEAT-20.SPEC-001 performing the initialisation in the same step that creates the account, per the Defaults and Derivations table.

**FEAT-20.SPEC-005-AC-14:** Given onboarding_status transitions to Complete, when the transition occurs, then onboarding_completed_at is set once to that moment and is never subsequently changed.

**FEAT-20.SPEC-005-AC-15:** Given Nadia's project creation fails inside the "Add first client and project" step, when she returns to FEAT-20.SPEC-002, then the step stays current marked "Not finished" with a "Try again" action, no "continue anyway" is offered, nothing is added to onboarding_failed_incomplete_steps, and onboarding_status stays In Progress, per Rule R-04.

**FEAT-20.SPEC-005-AC-16:** Given Nadia opens the "How did you hear" question and closes the browser without answering, when she signs in later, then onboarding_how_did_you_hear_resolved is still false and the question is shown again, per Rule R-06.

**FEAT-20.SPEC-005-AC-17:** Given Nadia answers or skips the question, when the answer or skip is saved, then FEAT-20.SPEC-002 only triggers FEAT-20.SPEC-004, and FEAT-20.SPEC-004 alone sets onboarding_how_did_you_hear_resolved true after its hand-off attempt (true even if the hand-off fails) and not before, per Rule R-06.

**FEAT-20.SPEC-005-AC-18:** Given a new Freelancer Account arrived via a FEAT-33 mark, when it is created, then onboarding_referring_portal_ref holds that reference and never changes afterward.

**FEAT-20.SPEC-005-AC-19:** Given onboarding is In Progress, when Nadia opens the dashboard or any other feature directly, then it opens normally with no gate, redirect, or hidden content, and her next sign-in lands on FEAT-20.SPEC-002 at her current step, per Rule R-08.

**FEAT-20.SPEC-005-AC-20:** Given onboarding_status becomes Complete while onboarding_how_did_you_hear_resolved is false, when the transition occurs, then the account is auto-resolved as "unknown", the FEAT-20.SPEC-004 hand-off fires once with the persisted referring-portal reference, and the question is never shown, per Rule R-09.

**FEAT-20.SPEC-005-AC-21:** Given onboarding_status is Complete and Nadia has tapped "Go to your dashboard", when she later opens FEAT-20.SPEC-002's URL or signs in, then she lands on the dashboard (FEAT-12) and the Ready state is not shown again, per Rule R-08.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 10 | 10 |
| Business Rules | 9 | 9 |
| Edge Cases | 9 | 9 |



# Notification Spec: Welcome Email

## Overview

**Name:** Welcome Email
**ID:** FEAT-20.SPEC-006
**Type:** Notification
**Purpose:** Confirms to Nadia that her Freelancer Account was created, using the transactional email delivery capability, so she has a record of her sign-up separate from the in-product session.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- The single welcome email sent once, at account creation
- Its delivery, retry, and expiry behavior on its one channel (email)

**Non-Goals:**
- Deciding that a Freelancer Account should be created -- owned by FEAT-20.SPEC-001 (Sign-Up & Account Creation); this spec begins where that screen's creation trigger fires
- Any onboarding-progress content (which step Nadia is on, what remains) -- this email confirms account creation only; onboarding's own progress is shown live inside FEAT-20.SPEC-002, not repeated in email form
- An opt-out control for this email -- excluded per product-features.md's notification_preferences definition: "transactional record emails cannot be disabled" (XBR-30), and confirming account creation is exactly this kind of transactional record
- General product-education or onboarding-tips email sequences -- product-features.md defines no drip-email capability; this spec covers only the single confirmation named in the Brief's Communications field

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, once, when the Freelancer Account is created | Nadia's sign-in email is the one contact point established at account creation, and the product's baseline notification design uses email for every transactional record (ASMP-26); no in-app equivalent is needed since Nadia is already looking at the guided sequence in-product at the same moment |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Freelancer Account created | FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Fires once, immediately after the Freelancer Account is created and transitioned to Active | Freelancer's name, sign-in email |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient, sent to the sign-in email address she just registered. No other role in the Access Matrix has any relationship to this email; Owen, Priya, and Dana never receive it.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional record email | N/A | Always sent | N/A -- per XBR-30, transactional emails core to the record always send and carry no opt-out; FEAT-21's notification-preference screen governs only optional emails, never this one |

**Quiet Hours:** N/A -- account creation is a one-time, freelancer-initiated moment (she is at that instant actively signing up), so there is no meaningful "wrong time" for this confirmation; the product defines no quiet-hours window for this email.

## Content Definition

**Email:**
- **Subject:** Welcome to Clientroom, {freelancer_name}
- **Body:**
  Hi {freelancer_name},

  Your Clientroom account is set up and ready. You're picking up right where you left off in your guided setup -- add your first client, draft your first proposal, and you're on your way to getting paid faster.

  If you didn't create this account, contact us right away.
- **CTA (button):** Continue setup -- deep-links to FEAT-20.SPEC-002 (Onboarding Guided Sequence), resuming on Nadia's current onboarding step

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_name} | Freelancer Account -- name | Jordan Lee | Never empty -- name is required at account creation (FEAT-20.SPEC-001) |

## Delivery Rules

**Batching:** N/A -- this email is sent exactly once per account; there is never a second pending instance to batch with.
**Deduplication:** At most one welcome email per Freelancer Account. A retried or repeated account-creation attempt that fails before the account is actually created (per FEAT-20.SPEC-001's Edge Cases) never triggers a send, since the trigger fires only on a successful, committed account creation.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning per ASMP-26's delivery-visibility expectation. FEAT-14.SPEC-006 defines the warning on the affected project, but no project exists yet at this moment, so the warning is shown in the notice area of the onboarding shell (FEAT-20.SPEC-002), which defines its surface, exact wording ("We couldn't deliver your welcome email to {sign_in_email}. If that address is wrong, you can correct it in Settings."), "Go to Settings" button (to FEAT-21.SPEC-005) and "Dismiss" link. It appears the next time Nadia opens the shell while onboarding is In Progress or on the Ready state, and stops appearing once she dismisses it or changes her sign-in email.
**Expiry:** This email never expires in the sense of becoming irrelevant to skip -- confirming account creation stays true indefinitely. If all retries are exhausted, delivery is not reattempted later, and the surviving signal is that Nadia's account and onboarding progress remain fully visible and usable in-product regardless of whether this email ever arrived.

## Edge Cases

- **Nadia's sign-in email is mistyped and does not exist** -- Delivery bounces; per the Retry rule, retries are attempted up to the standard count and window, then the failure is surfaced as a delivery warning inside the guided sequence, since correcting the email address is itself a Settings action (FEAT-21) available to her once she is signed in.
- **The Freelancer Account is deleted (FEAT-24) before this email is delivered** -- Delivery is cancelled; a welcome confirmation for an account that no longer exists is never sent.
- **Preferences change between trigger and delivery** -- Not applicable: this email carries no preference to change (Audience and Preferences).
- **Quiet hours** -- Not applicable, per the Channels/Quiet Hours definition above.
- **Delivery succeeds but arrives after Nadia has already completed onboarding** -- The email's content and CTA remain valid regardless of timing; the CTA opens FEAT-20.SPEC-002, which lands her on the Ready state if onboarding is Complete and she has not yet tapped "Go to your dashboard", or redirects her to the dashboard (FEAT-12) if she has, rather than mid-sequence.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Triggered by (inbound) | Account creation fires this email |
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Navigation (outbound) | The "Continue setup" CTA deep-links here, resuming Nadia's current step |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | This email is sent and its delivery/bounce status tracked through the transactional email delivery capability |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | References (outbound) | The project-level warning this account-level variant stands in for, because no project exists at sign-up |
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Affects (outbound) | Hosts the welcome-email delivery warning (surface, wording, dismissal) |
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Navigation (outbound, via the warning) | The warning's "Go to Settings" button leads here to correct a mistyped address |

## Analytics and Success Signals

- **welcome_email_delivered** (delivery outcome: delivered / bounced / failed) -- supports success-metrics.md: "First-Session Activation" (a delivered welcome confirmation is part of the trusted first-session experience the metric measures)
- **welcome_email_delivery_warning_shown** (retries exhausted) -- N/A -- no Stage 2 metric measures individual delivery-warning views; retained per ASMP-26's delivery-visibility expectation so a lost first email is observable rather than invisible

## Acceptance Criteria

**FEAT-20.SPEC-006-AC-01:** Given Nadia successfully creates her Freelancer Account, when the account is created, then the welcome email is sent once to her sign-in email address with the subject "Welcome to Clientroom, {freelancer_name}."

**FEAT-20.SPEC-006-AC-02:** Given Nadia receives the welcome email, when she taps "Continue setup," then she lands on FEAT-20.SPEC-002 (Onboarding Guided Sequence) at her current onboarding step.

**FEAT-20.SPEC-006-AC-03:** Given Nadia's sign-in email address does not exist, when delivery is attempted, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears to her inside the guided sequence.

**FEAT-20.SPEC-006-AC-04:** Given Nadia's Freelancer Account is deleted before this email is delivered, when the deletion completes, then delivery of this email is cancelled.

**FEAT-20.SPEC-006-AC-05:** Given Nadia already completed onboarding by the time this email is delivered, when she taps "Continue setup," then she lands on the Ready state if she has not yet tapped "Go to your dashboard", or is redirected to her dashboard (FEAT-12) if she has, rather than mid-sequence, per FEAT-20.SPEC-002.

**FEAT-20.SPEC-006-AC-06:** Given this feature defines no preference control for this email, when Nadia looks in her notification preferences (FEAT-21), then no toggle for this email exists there -- it always sends.

**FEAT-20.SPEC-006-AC-07:** Given Nadia's account creation attempt fails validation and no account is created, then no welcome email is sent, since the trigger fires only on a successful, committed account creation.

**FEAT-20.SPEC-006-AC-08:** Given delivery succeeds on the first attempt, when it is delivered, then `welcome_email_delivered` is emitted with outcome "delivered."

**FEAT-20.SPEC-006-AC-09:** Given all retries are exhausted with no successful delivery, when the final retry fails, then the next time Nadia opens FEAT-20.SPEC-002 the warning is shown in its notice area with "Go to Settings" and "Dismiss", and `welcome_email_delivery_warning_shown` is emitted at that display.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always sent, no preference) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
