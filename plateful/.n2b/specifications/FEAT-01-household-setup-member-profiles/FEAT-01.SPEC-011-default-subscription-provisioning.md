---
document_type: spec
spec_type: automation
spec_id: FEAT-01.SPEC-011
spec_name: Default Subscription Provisioning
spec_slug: default-subscription-provisioning
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 6
---

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
