---
document_type: spec
spec_type: automation
spec_id: FEAT-23.SPEC-002
spec_name: Free Plan Auto-Provisioning
spec_slug: free-plan-auto-provisioning
parent_feature: FEAT-23
parent_feature_name: Subscription Plan & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Free Plan Auto-Provisioning

## Overview

**Name:** Free Plan Auto-Provisioning
**ID:** FEAT-23.SPEC-002
**Type:** Automation
**Purpose:** Creates the Subscription Plan record on the free tier automatically the instant a new Freelancer Account is created, so there is never an explicit "no plan" state.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Creating exactly one Subscription Plan record the moment a Freelancer Account is created
- Setting that record's initial tier, status, and billing_cycle per FEAT-23.SPEC-007's defaults
- Handling the failure path if provisioning itself cannot complete, including stopping when the account is being deleted
- Reporting the created plan to the Activity & Audit Trail (FEAT-13.SPEC-003)

**Non-Goals:**
- Any subsequent tier or status change (upgrade, downgrade, cancellation, lapse) -- owned by FEAT-23.SPEC-004 (Plan State Sync); this automation runs exactly once, at creation.
- Validating or authorizing the resulting record's fields -- owned by FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules); this automation only applies the defaults that spec defines.
- The account-creation flow itself (sign-up form, credential capture) -- owned by FEAT-20.SPEC-001 (Sign-Up & Account Creation); this automation begins only after that account is created.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| New Freelancer Account created | FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Always -- fires exactly once, the instant a new Freelancer Account record is committed | Freelancer Account reference |
| Account deletion begins (hold phase) or completes | FEAT-24.SPEC-004 (Account Deletion Processing) | Fires only when the account is marked pending-delete or hard-deleted while provisioning has not yet succeeded | Freelancer Account reference, phase (hold / deleted) |

## Processing Logic

1. Receive the new Freelancer Account reference from account creation (FEAT-20.SPEC-001).
2. Create one Subscription Plan record linked one-to-one to that Freelancer Account.
3. Set tier to Free, status to Active, and billing_cycle to unset, per FEAT-23.SPEC-007's Defaults and Derivations.
4. Confirm the record was created before the onboarding sequence (FEAT-20.SPEC-002) proceeds -- there is no intermediate "no plan" state visible to Nadia at any point.
5. Make the new record available for the first read by FEAT-23.SPEC-001 (Plan & Billing Screen) and by FEAT-01.SPEC-008 (Active Client Limit Enforcement) the moment the account exists.
6. After the record is committed, report the creation as a record-worthy event (event type plan created, actor "Automatic", tier=Free, status=Active, timestamp) to FEAT-13.SPEC-003 (Activity Entry Recording) for the append-only trail. The trail write is FEAT-13's responsibility and retries there; it never delays or reverses the plan record or the onboarding sequence.
7. If the account deletion trigger (FEAT-24.SPEC-004) arrives before provisioning has succeeded, stop retrying and create no record; if it arrives after, do nothing here -- the plan record is removed by FEAT-24.SPEC-004 itself and this automation never runs again for that account.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Plan provisioned | Freelancer Account creation commits successfully | Subscription Plan record created: tier=Free, status=Active, billing_cycle=unset, linked to the new account | None distinct -- Nadia's onboarding sequence and Plan & Billing Screen simply reflect the free tier from her first view; no separate confirmation of provisioning is shown | FEAT-20.SPEC-002 (Onboarding Guided Sequence), FEAT-23.SPEC-001 (Plan & Billing Screen), FEAT-01.SPEC-008 (Active Client Limit Enforcement) |
| Provisioning failure | The Subscription Plan record cannot be created immediately following a successful account creation | No Subscription Plan record exists yet; the Freelancer Account exists | Nadia's onboarding sequence and any screen reading her plan show a temporary "Setting up your plan -- try again in a moment" state rather than treating the account as planless; the automation retries automatically | FEAT-20.SPEC-002, FEAT-23.SPEC-001 |
| Provisioning stopped by account deletion | Account deletion hold or completion arrives before provisioning has succeeded | No Subscription Plan record is created; the retry stops | None -- the account is being deleted, so no plan is needed; FEAT-24.SPEC-004 owns the deletion experience | FEAT-24.SPEC-004 |

## Data Model

**Reads:** Freelancer Account -- the reference to the newly created account only (no other account fields).
**Creates:** Subscription Plan -- tier, status, and billing_cycle set per FEAT-23.SPEC-007's defaults; linked one-to-one to the Freelancer Account.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Exactly one Subscription Plan record exists per Freelancer Account at all times after this automation completes (Relationships field, feature-dependency-map.md) -- this automation is the entity's sole creation path.
- This automation is non-blocking toward account creation itself: the Freelancer Account is already committed by the time this automation runs, so a provisioning failure never undoes or blocks the sign-up that just completed (FEAT-20.SPEC-001).
- Provisioning is automatic and requires no action or acknowledgement from Nadia -- there is no "choose your starting plan" step; the free tier is the only possible starting state (product-features.md, States field: "a brand-new account starts on the free tier automatically, no explicit 'no plan' state").

## Edge Cases

- **Account creation succeeds but provisioning fails immediately after** -- The automation retries automatically. Until it succeeds, any screen that would read the plan (FEAT-23.SPEC-001, FEAT-01.SPEC-008) shows "Setting up your plan -- try again in a moment" rather than a blank or error state; no client-limit gate can be evaluated meaningfully until the plan exists, so client-adding is temporarily unavailable with the same message.
- **Two account-creation attempts for the same freelancer occur near-simultaneously (e.g., a double form submission)** -- Account creation itself (FEAT-20.SPEC-001) permits only one Freelancer Account per sign-up, so this automation never receives two triggers for what becomes one account; if it somehow did, the one-per-account relationship is enforced by rejecting a second Subscription Plan creation for an account that already has one.
- **The account is deleted (FEAT-24.SPEC-004) while provisioning is still retrying** -- The retry stops and no plan is ever created; if the plan was created just before deletion began, the record is removed by the account deletion process, not by this automation.
- **Concurrent trigger firing (two new accounts created at effectively the same time)** -- Each trigger carries its own distinct Freelancer Account reference, so each provisions its own independent Subscription Plan record; there is no shared state between the two runs.
- **Trigger fires while a previous run is in flight** -- Not applicable: each run is scoped to a single, already-unique Freelancer Account, and account creation produces exactly one trigger per account, so no second run for the same account can ever be in flight concurrently with the first.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Triggered by (inbound) | New Freelancer Account creation fires this automation |
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Affects (outbound) | The guided sequence reads the provisioned plan (or the temporary setup state) from its first step |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Affects (outbound) | Displays the newly provisioned Free-tier plan on first view |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement) | Affects (outbound) | Reads the provisioned plan's tier and status to gate the first client add |
| FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules) | References (inbound) | Supplies the default values this automation applies |
| FEAT-13.SPEC-003 (Activity Entry Recording, FEAT-13) | Triggers (outbound) | The created plan is reported as an append-only trail entry (Brief Cross-Feature Touchpoint); FEAT-13.SPEC-003's own trigger list is owned by FEAT-13 |
| FEAT-24.SPEC-004 (Account Deletion Processing, FEAT-24) | Triggered by (inbound) | Account deletion during a pending provisioning retry stops it; the plan record itself is deleted by FEAT-24 |

## Analytics and Success Signals

- **plan_provisioned** (tier: free) -- N/A -- no Stage 2 metric measures provisioning itself; success-metrics.md's Free-to-Paid Conversion metric measures the later upgrade decision (fed by FEAT-23.SPEC-004), not the automatic starting state.
- **plan_provisioning_failed** (retry_attempt) -- N/A -- no Stage 2 metric covers this internal reliability signal; retained so a stuck provisioning path is observable rather than silently blocking onboarding.

## Acceptance Criteria

**FEAT-23.SPEC-002-AC-01:** Given Nadia completes sign-up (FEAT-20.SPEC-001), when her Freelancer Account is created, then a Subscription Plan record is created immediately with tier=Free, status=Active, and billing_cycle unset.

**FEAT-23.SPEC-002-AC-02:** Given Nadia's account was just created, when she opens the Plan & Billing Screen (FEAT-23.SPEC-001) for the first time, then it shows the Free tier -- never an empty or "no plan" state.

**FEAT-23.SPEC-002-AC-03:** Given Nadia's account was just created, when she attempts to add her first client (FEAT-01.SPEC-008), then the free-tier limit check evaluates immediately against her already-provisioned plan.

**FEAT-23.SPEC-002-AC-04:** Given account creation succeeds, when the immediate provisioning attempt fails, then Nadia's onboarding sequence and plan screen show "Setting up your plan -- try again in a moment" and the automation retries automatically.

**FEAT-23.SPEC-002-AC-05:** Given the provisioning retry succeeds after an initial failure, when Nadia next views her plan, then it shows the Free tier normally with no trace of the earlier failure.

**FEAT-23.SPEC-002-AC-06:** Given two new Freelancer Accounts are created at effectively the same time, when this automation fires for each, then each account receives its own independent Subscription Plan record with no interference between the two.

**FEAT-23.SPEC-002-AC-07:** Given a Freelancer Account already has a provisioned Subscription Plan, when a duplicate provisioning attempt is made for that same account, then it is rejected and the existing record is left unchanged.

**FEAT-23.SPEC-002-AC-08:** Given Nadia's account is newly created, when any other feature (FEAT-01, FEAT-16) reads her plan before she has taken any billing action, then it reads Free/Active with no distinction from a plan she might have actively chosen.

**FEAT-23.SPEC-002-AC-09:** Given Nadia's plan record was just created, when the commit completes, then a plan-created event (tier Free, status Active, actor "Automatic") is reported to FEAT-13.SPEC-003, and a trail-write retry never delays or reverses the plan or her onboarding.

**FEAT-23.SPEC-002-AC-10:** Given provisioning is still retrying after an initial failure, when Nadia's account deletion begins or completes (FEAT-24.SPEC-004), then the retry stops and no Subscription Plan record is created.

**FEAT-23.SPEC-002-AC-11:** Given Nadia's plan was provisioned before her account deletion began, when deletion completes, then the record is removed by FEAT-24.SPEC-004 and this automation does not run again for that account.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 3 | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
