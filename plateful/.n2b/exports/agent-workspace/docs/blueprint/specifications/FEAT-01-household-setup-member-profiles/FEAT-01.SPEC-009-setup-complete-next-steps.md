---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-009
spec_name: Setup Complete & Next Steps
spec_slug: setup-complete-next-steps
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

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
