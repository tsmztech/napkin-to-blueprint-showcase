---
document_type: spec
spec_type: automation
spec_id: FEAT-24.SPEC-005
spec_name: Referral Upgrade Tracking
spec_slug: referral-upgrade-tracking
parent_feature: FEAT-24
parent_feature_name: Invite Another Household
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 7
---

# Automation Spec: Referral Upgrade Tracking

## Overview

**Name:** Referral Upgrade Tracking
**ID:** FEAT-24.SPEC-005
**Type:** Automation
**Purpose:** Sets a recorded Household Referral's `upgraded` flag when the referred household's own Subscription becomes paid.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- Detecting when a referred household's Subscription transitions to paid
- Updating the matching Household Referral record's `upgraded` field to true
- Leaving `upgraded` at false for a referred household that never upgrades, or that upgrades and later reverts to free

**Non-Goals:**
- Deciding whether a household's Subscription becomes paid, or processing the payment itself -- owned entirely by FEAT-14 (Subscription & Billing Management); this automation only reads the outcome
- Creating the Household Referral record itself -- owned by FEAT-24.SPEC-004 (Household Referral Recording); this automation only updates an already-existing record
- Reverting `upgraded` back to false if the household later downgrades or cancels -- excluded per the dependency map's Household Referral entity definition, which lists `upgraded` as "whether the new household went on to pay" without a stated downgrade-reversal behavior; once true, the flag reflects that the household did go on to pay at least once, which is the fact the growth metric depends on
- Any reward, credit, or discount tied to a referred household's upgrade -- excluded per scope-boundaries.md SC-10: no money passes between households in this feature

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household's subscription tier changes to paid | FEAT-14.SPEC-008 (Apply Subscription Change), "Upgrade applied" outcome | Fires whenever a household's Subscription tier is set to paid, for any household in the product | Household reference whose Subscription changed, new tier (paid) |

## Processing Logic

1. Receive the Household reference whose Subscription just changed to paid.
2. Check whether this Household is the `new_household` on any existing Household Referral record.
3. If no matching record exists, stop -- this household was never referred, or its referral fell outside the eligibility rules at creation time (FEAT-24.SPEC-006), so there is nothing to update.
4. If a matching record exists and its `upgraded` field is already true, stop -- no change is needed.
5. If a matching record exists and its `upgraded` field is false, set it to true.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Upgrade recorded | The subscribing household has a matching Household Referral record with `upgraded` currently false | Household Referral -- `upgraded` set to true | None directly visible on any screen; the referring household's derived paying share (FEAT-24.SPEC-001, Data Notes) reflects the change on next view | FEAT-24.SPEC-001 |
| Already recorded | A matching record exists and `upgraded` is already true | None | None | -- |
| No matching referral | The subscribing household was never a referred household | None | None -- this is the ordinary path for the large majority of subscription upgrades, which have no referral to update | -- |
| Automation failure | An internal processing error prevents the update | Household Referral record's `upgraded` field retains its prior value | None user-visible -- FEAT-14's own upgrade confirmation (FEAT-14.SPEC-010) is unaffected, since this automation's failure never blocks or alters the subscription change itself | -- |

## Data Model

**Reads:** Household Referral -- existing records, matched by new_household, to find the one (if any) belonging to the subscribing household; Subscription -- read indirectly through FEAT-14.SPEC-008's trigger, which supplies the tier change itself rather than requiring a separate read here.
**Creates:** None.
**Updates:** Household Referral -- `upgraded` field only, set from false to true. This is the only field this feature ever updates on the record (Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix).
**Deletes:** None.

## Business Rules

- This automation is the exclusive writer of the Household Referral's `upgraded` field -- no screen or role sets it directly (FEAT-24.SPEC-006, Authorization Rules).
- A household's Subscription change never waits on this automation -- FEAT-14's own upgrade flow and confirmation are entirely unaffected by whether a matching referral exists or by this automation's outcome, since referral tracking is a side effect, not a precondition, of any subscription change (mirrors FEAT-24.SPEC-004's equivalent rule for household creation).
- Once `upgraded` is set to true, it is never reverted, per this feature's decision authority: the dependency map's Household Referral entity defines no downgrade-reversal behavior, and the record functions as the product's own history of referral-driven growth (dependency map, Household Referral Data Sensitivity), not a live subscription-status mirror.

## Edge Cases

- **Household upgrades, downgrades, and upgrades again** -- The `upgraded` flag is already true from the first upgrade and needing no further change; the second upgrade's trigger reaches Step 4 of Processing Logic and stops as "already recorded," leaving the flag as-is.
- **Household referral record does not yet exist when the subscription upgrade fires (a rare ordering case where FEAT-24.SPEC-004 has not yet completed for a household that upgrades within moments of completing its own setup)** -- No matching record is found at this moment, so no update occurs and none is queued; since Household Referral records are created once, synchronously, at household completion (FEAT-24.SPEC-004), and a paid-tier upgrade cannot occur before a household exists, this ordering is not reachable in practice, but the "no matching referral" outcome handles it safely if it ever occurred.
- **Concurrent trigger firing (two different referred households upgrade to paid at effectively the same time)** -- Each triggers its own independent evaluation against its own Household Referral record; the two updates target different records and do not interact.
- **Trigger fires while a previous run is in flight for the same household (e.g., a duplicate upgrade signal from FEAT-14.SPEC-008 after a retried payment confirmation)** -- Step 4's already-true check makes a second run for the same household a no-op once the first run has completed; if both runs somehow evaluate before either writes, the update itself (setting `upgraded` to true) is idempotent, so no inconsistent state results.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggered by (inbound) | The "Upgrade applied" outcome fires this automation |
| FEAT-24.SPEC-004 (Household Referral Recording) | References (inbound) | Supplies the Household Referral record this automation updates |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Confirms `upgraded` is the only field this feature updates on the record, and that this automation is its exclusive writer |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Affects (outbound) | The referring household's derived paying share reflects an updated `upgraded` flag |

## Analytics and Success Signals

- **referred_household_upgraded** (days_since_referral_created: derived) -- supports success-metrics.md: "Household-to-Household Invitation Growth"

## Acceptance Criteria

**FEAT-24.SPEC-005-AC-01:** Given a referred household's Subscription changes to paid, when this automation fires, then the matching Household Referral record's `upgraded` field is set to true.

**FEAT-24.SPEC-005-AC-02:** Given a household's Subscription changes to paid but the household was never referred, when this automation fires, then no Household Referral record is found or updated.

**FEAT-24.SPEC-005-AC-03:** Given a referred household's `upgraded` flag is already true, when its Subscription changes to paid again (e.g., after a re-subscription), then this automation makes no change, since the flag is already true.

**FEAT-24.SPEC-005-AC-04:** Given a referred household upgrades, later downgrades to free, when this automation is inspected after the downgrade, then the `upgraded` flag remains true, since it is never reverted.

**FEAT-24.SPEC-005-AC-05:** Given two different referred households upgrade to paid at effectively the same time, when this automation fires for each, then each household's own Household Referral record is updated independently and correctly.

**FEAT-24.SPEC-005-AC-06:** Given a duplicate upgrade signal arrives for a household whose referral record was just updated, when this automation runs a second time, then no error occurs and the flag remains true.

**FEAT-24.SPEC-005-AC-07:** Given an internal processing error prevents this automation's update, when the failure occurs, then FEAT-14's own upgrade confirmation to the referred household proceeds unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (recorded, already recorded, no matching referral, automation failure) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
