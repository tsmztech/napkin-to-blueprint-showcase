---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-20.SPEC-005
spec_name: Onboarding Step Sequencing & Exit-Criteria Rules
spec_slug: onboarding-step-sequencing-exit-criteria-rules
parent_feature: FEAT-20
parent_feature_name: Onboarding / First-Run Setup
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 21
---

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
