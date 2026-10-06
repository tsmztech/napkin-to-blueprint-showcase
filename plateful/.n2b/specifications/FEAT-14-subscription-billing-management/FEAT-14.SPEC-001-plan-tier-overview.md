---
document_type: spec
spec_type: screen
spec_id: FEAT-14.SPEC-001
spec_name: Plan Tier Overview
spec_slug: plan-tier-overview
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Plan Tier Overview

## Overview

**Name:** Plan Tier Overview
**ID:** FEAT-14.SPEC-001
**Type:** Screen
**Purpose:** Shows the household's current plan tier and what each tier includes, and is the entry point Maya uses to reach every other billing action.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Displaying the household's current tier (free or paid) and what each tier includes
- The persistent grace/billing-state banner when billing_state is not Active, or when a change (downgrade or cancellation) is pending against the current period even while billing_state remains Active
- Entry points to Upgrade, Manage Billing, and Downgrade/Cancel, visible only to Maya

**Non-Goals:**
- Collecting payment details or completing an upgrade -- owned by FEAT-14.SPEC-002 (Upgrade to Paid); this screen only offers the entry point
- Updating payment details, viewing billing history, or switching billing period -- owned by FEAT-14.SPEC-003 (Billing & Payment Management)
- Downgrading or cancelling -- owned by FEAT-14.SPEC-004 (Downgrade / Cancel)
- Deciding who may view or act on this screen -- the exact rules are governed by FEAT-14.SPEC-006 (Tier & Billing Access Authorization); this screen enforces but does not redefine them

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | Maya taps "See plans" at the end of guided setup | None -- this screen loads the household's just-provisioned free-tier Subscription |
| External / default entry | Any household member opens the product's persistent plan/billing entry point | None -- the screen always loads the household's current Subscription. This is a distinct, always-available entry point from FEAT-01.SPEC-010 (Household Settings Hub), which does not expose billing (per that spec's own scope) |
| FEAT-14.SPEC-002, FEAT-14.SPEC-003, FEAT-14.SPEC-004 | Maya taps back, or completes/cancels an action on any billing screen | Updated tier and billing_state reflecting the just-applied change |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Maya taps "View plan" | None -- loads the household's current Subscription |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen, including tier, inclusions, and the billing-state banner | All actions: Upgrade, Manage Billing, Downgrade/Cancel entry points | -- |
| Sam (Other Adult Member) | Tier, inclusions, and the billing-state banner (every adult member can see which tier the household is on) | None -- no billing actions shown | Upgrade, Manage Billing, and Downgrade/Cancel entry points are not shown; there is no action to deny |
| Jordan (young kid profile, no login -- MVP) | No | No | Screen is unreachable -- a no-login profile has no sign-in path to any screen |
| Jordan (older kid, limited login -- Later) | No | No | Entry points to this screen are not shown; a direct link resolves to "This isn't part of your household view." per the older-kid row's Billing: None |
| Riley (Operator, support -- from v1) | Plan tier only, never payment details or billing history, and only while an open Support Request exists for this household (XBR-14) | No actions | Outside an open Support Request, the screen is unreachable to Riley; when reachable, no action controls render at all |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on this screen only if they are a household member entitled to view it |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; no in-progress input exists on this read-only screen, so nothing needs to be preserved |

## Layout and Content

**Header:** Screen title "Plan & Billing" with a back arrow (returns to wherever the member entered from).

**Body:**
- **Current Tier card**, at the top: the household's tier ("Free" or "Paid -- Monthly" / "Paid -- Yearly"), non-interactive.
- **Billing-state banner**, directly below the tier card, shown when billing_state is not Active, or when pending_change is not none even while billing_state is Active (a pending downgrade): "Payment failed -- update your card by {payment_failure_date + 7 days}." for billing_state Payment failed; "Your paid features continue until {current_period_end_date}, then the household moves to the free tier." for billing_state Cancelled; "Your household moves to the free tier on {current_period_end_date}. Paid features stay active until then." for billing_state Active with pending_change downgrade. Same wording source as FEAT-14.SPEC-003's identical banner, per the Feature Breakdown Brief's Shared UI Patterns.
- **Tier-inclusion summary block**: a two-column comparison ("Free" / "Paid") listing what each tier includes -- manual weekly planning and the shared grocery list on Free; the AI-generated weekly plan, pantry-aware plan weighting, and rating-based learning added on Paid. Identical content to the block shown on FEAT-14.SPEC-002 and FEAT-14.SPEC-004, per the Brief's Shared UI Patterns; only the surrounding action differs here.
- **Action row** (Maya only): "Upgrade" button, shown only when tier is free; "Manage Billing" button, shown only when tier is paid; "Downgrade" or "Cancel" links, shown only when tier is paid, inside the Manage Billing area they lead to.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Tier card, banner, inclusion comparison, and action row stack vertically, full width, one-thumb reachable per assumptions-constraints.md ASMP-29.
- **Medium size class and above:** The tier-inclusion comparison renders as two side-by-side columns instead of stacked; the tier card and banner remain full width above it.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry source | Screen closes | Standard transition |
| Current Tier card | -- | Display-only, non-interactive | None | -- |
| Billing-state banner | Tap (Maya only) | Navigate to FEAT-14.SPEC-003 (Billing & Payment Management) | Screen changes | Standard transition |
| Billing-state banner (Sam, Riley) | Tap | No navigation -- banner is display-only for these roles | None | -- |
| Tier-inclusion summary block | -- | Display-only, non-interactive | None | -- |
| "Upgrade" button (Maya, free tier only) | Tap | Navigate to FEAT-14.SPEC-002 (Upgrade to Paid) | Screen changes | Standard transition |
| "Manage Billing" button (Maya, paid tier only) | Tap | Navigate to FEAT-14.SPEC-003 (Billing & Payment Management) | Screen changes | Standard transition |

### Accessibility Notes

- **Focus order:** Back arrow -> Current Tier card -> Billing-state banner (when shown) -> Tier-inclusion summary block -> Upgrade or Manage Billing button (when shown).
- **Dynamic announcements:** When the billing-state banner appears or its wording changes (e.g., grace period counting down), the update is announced to assistive technology as a live-region change.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Tier card, inclusion summary, and (if applicable) banner and actions render normally | Screen opens and the household's Subscription loads successfully | User navigates away |
| Loading | Tier card and inclusion block show a loading placeholder; no actions are shown yet | Screen first opens, before the Subscription record has loaded | Load completes (success or error) |
| Error | Banner: "Couldn't load your plan. Check your connection and try again." with a Retry button; no tier or actions shown | Loading the Subscription record fails | User taps Retry and the load succeeds |
| Offline/Degraded | The last successfully loaded tier, inclusions, and billing-state banner remain visible with a banner: "You're offline -- showing your plan as of your last visit." Upgrade, Manage Billing, and Downgrade/Cancel actions are disabled with the note "Requires a connection." | Connectivity is lost while viewing, or the screen is opened with no connectivity but a cached tier exists | Connectivity is restored -- the screen refreshes silently and re-enables actions |

## Validation Rules

Validation governed by FEAT-14.SPEC-006 (Tier & Billing Access Authorization). See that spec for who may view and act on this screen. This screen has no user-input fields of its own.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | Entry source (FEAT-01.SPEC-009 or the product's persistent navigation) | -- |
| "Upgrade" tap (Maya) | FEAT-14.SPEC-002 (Upgrade to Paid) | -- |
| "Manage Billing" tap (Maya) | FEAT-14.SPEC-003 (Billing & Payment Management) | -- |
| Billing-state banner tap (Maya) | FEAT-14.SPEC-003 (Billing & Payment Management) | -- |

## Data Model

**Creates:** None.
**Reads:** Subscription -- tier, billing_period, billing_state, pending_change, current_period_end_date, payment_failure_date (all feed the tier card and banner displayed on this screen). Household -- currency (to render tier pricing consistently, per ASMP-28), organiser (to determine whether the viewer is Maya for action visibility).
**Updates:** None.
**Deletes:** None.

## Business Rules

- Access to this screen and its actions is governed entirely by FEAT-14.SPEC-006 (Tier & Billing Access Authorization) -- this screen never independently decides who sees what.
- XBR-05: the tier-inclusion summary always reflects that manual planning, ratings, pantry logging, and the shared list stay available on Free; only AI generation, pantry-weighted suggestions, and rating-based learning are gated to Paid.
- The billing-state banner's wording is sourced identically here and on FEAT-14.SPEC-003, per the Brief's Shared UI Patterns -- neither screen defines its own variant wording; both screens drive the banner off the same billing_state and pending_change fields from FEAT-14.SPEC-005's Governed Entity.

## Edge Cases

- **Billing state changes while this screen is open (e.g., a renewal payment fails elsewhere in the same session)** -- The screen is live-updating: the tier card and banner refresh to reflect the new billing_state without requiring a manual reload. This is a display refresh, not a save conflict, since this screen never writes to Subscription.
- **Household viewed immediately after downgrade or grace-expiry reversion completes** -- The screen shows Free tier and no banner (billing_state is Active on the free tier after reversion), consistent with FEAT-14.SPEC-008's completed state.
- **Riley opens this screen outside any open Support Request** -- The screen is unreachable; Riley's support tooling routes only to households with an open request, per XBR-14.
- **Sam taps the billing-state banner** -- No navigation occurs; the banner is informational only for roles without Billing access.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | Navigation (inbound) | "See plans" hands off to this screen |
| FEAT-14.SPEC-002 (Upgrade to Paid) | Navigation (outbound) | "Upgrade" leads here |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Navigation (outbound) | "Manage Billing" and the billing-state banner lead here |
| FEAT-14.SPEC-006 (Tier & Billing Access Authorization) | References (inbound) | Governs who may view and act on this screen |
| FEAT-14.SPEC-008 (Apply Subscription Change) | References (inbound) | Every tier or billing_state change this screen displays originates here |
| FEAT-22 (Operator Read-Only Support Access) | Navigation (inbound) | Riley reaches a read-only version of this screen's tier display only within an open Support Request |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| plan_tier_overview_viewed | current tier (free / paid), billing_state, viewer role | Screen finishes loading successfully | supports success-metrics.md: "Paid Conversion Rate" |
| upgrade_entry_point_tapped | current tier | Maya taps "Upgrade" | supports success-metrics.md: "Paid Conversion Rate" |
| manage_billing_entry_point_tapped | current tier, billing_state | Maya taps "Manage Billing" or the billing-state banner | supports success-metrics.md: "Paying Household Retention" |

## Acceptance Criteria

**FEAT-14.SPEC-001-AC-01:** Given Maya's household is on the free tier, when she opens this screen, then she sees "Free" as the current tier, the tier-inclusion comparison, and an "Upgrade" button.

**FEAT-14.SPEC-001-AC-02:** Given Maya's household is on the paid tier with billing_state Active, when she opens this screen, then she sees "Paid" with its billing period, no billing-state banner, and a "Manage Billing" button.

**FEAT-14.SPEC-001-AC-03:** Given Maya's household has billing_state Payment failed, when she opens this screen, then the billing-state banner reads "Payment failed -- update your card by {date}." and tapping it navigates to FEAT-14.SPEC-003.

**FEAT-14.SPEC-001-AC-04:** Given Sam opens this screen, when it loads, then he sees the same tier and inclusion information Maya sees, but no Upgrade, Manage Billing, or Downgrade/Cancel entry points appear anywhere on the screen.

**FEAT-14.SPEC-001-AC-05:** Given the older-kid limited login (Later) attempts to reach this screen, when the attempt is made, then no entry point to it is shown, and a direct link resolves to "This isn't part of your household view."

**FEAT-14.SPEC-001-AC-06:** Given Riley is diagnosing an open Support Request for a household, when Riley views this screen through support access, then only the plan tier is shown -- no billing history or payment details, and no action controls render.

**FEAT-14.SPEC-001-AC-07:** Given the Subscription record fails to load, when the screen attempts to load it, then the error banner "Couldn't load your plan. Check your connection and try again." appears with a Retry button.

**FEAT-14.SPEC-001-AC-08:** Given Maya loses connectivity while viewing this screen, when she taps "Manage Billing", then the button is disabled with "Requires a connection." and the last-loaded tier remains visible.

**FEAT-14.SPEC-001-AC-09:** Given a household's Subscription reverts to free at grace-period expiry while Maya has this screen open, when the reversion completes, then the tier card updates to "Free" and the billing-state banner disappears without a manual reload.

**FEAT-14.SPEC-001-AC-10:** Given an unauthenticated visitor opens a link to this screen, when the link resolves, then they are redirected to the sign-in screen.

**FEAT-14.SPEC-001-AC-11:** Given Maya's session expires while this screen is open, when she attempts any action, then the dialog "Your session has expired. Sign in to continue." appears.

**FEAT-14.SPEC-001-AC-12:** Given Maya's household has billing_state Cancelled with current_period_end_date March 14, when she opens this screen, then the billing-state banner reads "Your paid features continue until March 14, then the household moves to the free tier."

**FEAT-14.SPEC-001-AC-13:** Given Maya's household has billing_state Active with pending_change downgrade and current_period_end_date March 14, when she opens this screen, then the billing-state banner reads "Your household moves to the free tier on March 14. Paid features stay active until then."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 4 (loaded, loading, error, offline) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
