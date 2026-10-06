---
document_type: feature-overview
feature_number: FEAT-20
feature_name: Onboarding / First-Run Setup
feature_slug: onboarding-first-run-setup
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 2
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 1
---

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
