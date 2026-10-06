# FEAT-14 — Subscription & Billing Management

This chapter covers FEAT-14, Subscription & Billing Management, a Important-tier feature. It contains 12 specifications carrying 151 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-14.SPEC-001 | Plan Tier Overview | screen | 13 |
| FEAT-14.SPEC-002 | Upgrade to Paid | screen | 12 |
| FEAT-14.SPEC-003 | Billing & Payment Management | screen | 14 |
| FEAT-14.SPEC-004 | Downgrade / Cancel | screen | 12 |
| FEAT-14.SPEC-005 | Billing State & Refund Rules | logic-rule | 16 |
| FEAT-14.SPEC-006 | Tier & Billing Access Authorization | logic-rule | 12 |
| FEAT-14.SPEC-007 | Payment Failure & Grace Period Handling | automation | 11 |
| FEAT-14.SPEC-008 | Apply Subscription Change | automation | 14 |
| FEAT-14.SPEC-009 | Payment Processing Integration | integration | 14 |
| FEAT-14.SPEC-010 | Billing Confirmation Notification | notification | 12 |
| FEAT-14.SPEC-011 | Payment Failure Grace-Period Notice | notification | 10 |
| FEAT-14.SPEC-012 | Transactional Email Delivery (Billing) | integration | 11 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Subscription & Billing Management

## Summary

**Feature:** Subscription & Billing Management
**ID:** FEAT-14
**Description:** The household can see its current plan tier, upgrade to the paid subscription to unlock the AI weekly plan and pantry-aware suggestions, and manage billing.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** The brief states the business model directly: a free tier for manual planning and the shared list, and "a paid household subscription (monthly or yearly)" that adds the AI plan, pantry suggestions, and learning (BRIEF.md, Business Context). Included at MVP because the product cannot generate any revenue, and the founder needs "paying households within about three months" (BRIEF.md, Constraints: Team/timeline), without it. [RESEARCH-INFORMED: added the finding that freemium with a low-cost annual household plan is the dominant model across all four profiled products, typically $10–$60 a year (vendor pricing pages, HIGH confidence)]

**Key Capabilities:**
- View current tier — Household sees whether it is on the free or paid tier and what each includes
- Upgrade to paid — Organiser subscribes monthly or yearly to unlock AI features
- Manage billing — Organiser updates payment details and views billing history
- Downgrade or cancel — Organiser can move back to the free tier, with a clear explanation of what is lost
- Switch between monthly and yearly — Organiser changes billing period, taking effect at the next renewal

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-14.SPEC-001 | Plan Tier Overview | Screen | Maya, Sam, Riley | Shows the household's current tier and what each tier includes; entry point to billing actions for Maya |
| FEAT-14.SPEC-002 | Upgrade to Paid | Screen | Maya | Organiser reviews what upgrading unlocks, chooses monthly or yearly, and enters payment details to subscribe |
| FEAT-14.SPEC-003 | Billing & Payment Management | Screen | Maya | Organiser updates payment details, views billing history, and switches billing period |
| FEAT-14.SPEC-004 | Downgrade / Cancel | Screen | Maya | Organiser reviews exactly what is kept and what is lost, then confirms a downgrade or cancellation |
| FEAT-14.SPEC-005 | Billing State & Refund Rules | Logic/Rule | Maya | Governs valid-payment requirements, downgrade/period-switch timing, the 7-day grace period, and the no-partial-refund and organiser-only money-flow constraints |
| FEAT-14.SPEC-006 | Tier & Billing Access Authorization | Logic/Rule | All | Enforces who can view tier status, who can change billing, and what an unauthorized visitor sees, per the Access Matrix |
| FEAT-14.SPEC-007 | Payment Failure & Grace Period Handling | Automation | Maya | On a renewal payment failure, opens a 7-day grace period without cutting off access, and reverts to free if it lapses unresolved |
| FEAT-14.SPEC-008 | Apply Subscription Change | Automation | Maya | Writes every tier, billing-period, and billing-state change to the Subscription record and signals the features that gate on it |
| FEAT-14.SPEC-009 | Payment Processing Integration | Integration | Maya | Product boundary to the payment-processing capability: submits payment methods, initiates and retries charges, and receives renewal outcome events |
| FEAT-14.SPEC-010 | Billing Confirmation Notification | Notification | Maya | Sends confirmation of an upgrade, downgrade, cancellation, or period switch to the organiser |
| FEAT-14.SPEC-011 | Payment Failure Grace-Period Notice | Notification | Maya | Sends the organiser a clear grace-period notice and a path to update payment details after a failed renewal |
| FEAT-14.SPEC-012 | Transactional Email Delivery (Billing) | Integration | Maya | Product boundary to the transactional email capability used to deliver billing confirmations and the grace-period notice |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| View current tier | FEAT-14.SPEC-001, FEAT-14.SPEC-006 | Overview screen displays tier and inclusions, gated by the access rule for who may see it | Phase 2 (Explicit) |
| Upgrade to paid | FEAT-14.SPEC-002, FEAT-14.SPEC-008, FEAT-14.SPEC-009 | Upgrade screen collects the plan choice and payment; the Integration submits the charge; Apply Subscription Change writes tier=paid and unlocks gated features | Phase 2 (Explicit) |
| Manage billing | FEAT-14.SPEC-003, FEAT-14.SPEC-009 | Billing screen updates payment details and shows billing history sourced from the Payment Processing Integration | Phase 2 (Explicit) |
| Downgrade or cancel | FEAT-14.SPEC-004, FEAT-14.SPEC-005, FEAT-14.SPEC-008 | Downgrade/Cancel screen explains what is kept and lost; the refund/timing rule sets when it takes effect; Apply Subscription Change executes it at period end | Phase 2 (Explicit) |
| Switch between monthly and yearly | FEAT-14.SPEC-003, FEAT-14.SPEC-005, FEAT-14.SPEC-008 | Period-switch action on the billing screen; the timing rule sets next-renewal effect; Apply Subscription Change executes it | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-14.SPEC-005 | Billing State & Refund Rules | Phase 5 (Rule Discovery) | The Validation & Limits field names five distinct conditions (valid payment, downgrade timing, 7-day grace, no partial refund, organiser-only money flow) shared across four specs — this exceeds the inline-validation threshold |
| FEAT-14.SPEC-006 | Tier & Billing Access Authorization | Phase 5 (Rule Discovery) | The Access field defines materially different behavior per role (Maya Full, Sam/kids None-but-tier-visible, Riley tier-only View, unauthorized visitor sees nothing) — conditional-by-role logic shared across screens, crossing the standalone-spec threshold |
| FEAT-14.SPEC-007 | Payment Failure & Grace Period Handling | Phase 4 (Trigger-Response + External Dependencies lens, inbound) | The Primary Flows & Alternates field's payment-failure and grace-expiry paths are cross-entity state transitions triggered by an inbound event from the payment-processing capability, not a direct data write |
| FEAT-14.SPEC-008 | Apply Subscription Change | Phase 4 (Trigger-Response) | Upgrade, period switch, downgrade, cancellation, and grace-expiry reversion all end in the same write to the Subscription record plus a signal to the features that gate on tier; factoring it out gives four entry points one shared write path instead of four divergent ones |
| FEAT-14.SPEC-009 | Payment Processing Integration | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-33) names payment processing as required for the paid subscription, and the dependency map's External Touchpoints table lists FEAT-14 as the only feature on that row, requiring its own Integration spec |
| FEAT-14.SPEC-010 | Billing Confirmation Notification | Phase 4 (Notification surfacing) | The Communications field names upgrade/downgrade confirmations as messages sent to the organiser, with real delivery rules (channel, audience, content) rather than a bare success toast |
| FEAT-14.SPEC-011 | Payment Failure Grace-Period Notice | Phase 4 (Notification surfacing) | The Communications field separately names the payment-failure grace-period notice, distinct in urgency and content from a routine confirmation, warranting its own Notification spec |
| FEAT-14.SPEC-012 | Transactional Email Delivery (Billing) | Phase 4 (External Dependencies lens) | assumptions-constraints.md's Dependencies section (ASMP-32) names transactional email as required for billing and grace-period notices, distinct from FEAT-01.SPEC-017's account/recovery route and FEAT-07.SPEC-006's plan-ready fallback route; each feature's email use case owns its own Integration spec per the established pattern |

## Entity-Lifecycle Coverage Matrix

**Entity: Subscription**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Created by FEAT-01 at household creation, defaulting to free (feature-dependency-map.md, Subscription Lifecycle: "Created by FEAT-01"); this feature never creates the entity, only transitions it | Explicit non-goal, not an omission — see Non-Goals |
| Read (single) | FEAT-14.SPEC-001, FEAT-14.SPEC-003 | Overview screen shows tier; Billing & Payment Management screen shows billing_period and billing_state detail | -- |
| Read (list) | FEAT-14.SPEC-003 | Billing history list, sourced from the Payment Processing Integration | -- |
| Update | FEAT-14.SPEC-008 | Apply Subscription Change writes tier, billing_period, and billing_state for every upgrade, period switch, downgrade, cancellation, and grace-expiry reversion | Entry points are FEAT-14.SPEC-002, SPEC-003, SPEC-004, and SPEC-007, which all route through SPEC-008 rather than writing directly |
| Delete/Archive | N/A | Subscription is never deleted directly by this feature; it is removed only through the Household deletion cascade owned by FEAT-18 (feature-dependency-map.md, Household Lifecycle: "Deleted by FEAT-18"). billing_history is retained for the life of the household account (scope-boundaries.md SC-18), with no purge | Explicit non-goal — see Non-Goals |
| State Transition | FEAT-14.SPEC-007, FEAT-14.SPEC-008 | billing_state transitions: Active → Payment failed (7-day grace) → Reverted to free (SPEC-007); Active → Cancelled → Reverted to free at period end (SPEC-008) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household | FEAT-14.SPEC-001, FEAT-14.SPEC-003 | Household's currency (ASMP-28) for displaying billing amounts and history; organiser identity for authorization |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Maya submits an upgrade (plan choice + payment) | Validate and submit payment through the Payment Processing Integration | Standalone Integration | FEAT-14.SPEC-009 |
| Payment for an upgrade succeeds | Apply the change: tier=paid, billing_state=Active; unlock AI plan generation, pantry-aware weighting, and rating-based learning | Standalone Automation | FEAT-14.SPEC-008 |
| Upgrade completes successfully | Send upgrade confirmation to Maya | Standalone Notification, delivered through Standalone Integration | FEAT-14.SPEC-010 / FEAT-14.SPEC-012 |
| Upgrade attempt fails (payment declined or processing error) | Preserve the chosen plan option on screen and offer a retry; no state change | Inline in triggering screen | FEAT-14.SPEC-002 |
| Maya switches billing period (monthly ↔ yearly) | Record the requested period; the timing rule sets it to take effect at next renewal | Standalone Logic/Rule + Standalone Automation | FEAT-14.SPEC-005 / FEAT-14.SPEC-008 |
| Period switch takes effect at renewal | Send confirmation of the new billing period to Maya | Standalone Notification | FEAT-14.SPEC-010 |
| Maya downgrades to free | Record the downgrade; the timing rule sets it to take effect at the end of the current paid period, never mid-period | Standalone Logic/Rule + Standalone Automation | FEAT-14.SPEC-005 / FEAT-14.SPEC-008 |
| Maya cancels | Same disposition as downgrade — paid features remain active until the already-paid period ends; no partial refund | Standalone Logic/Rule + Standalone Automation | FEAT-14.SPEC-005 / FEAT-14.SPEC-008 |
| Downgrade or cancellation takes effect at period end | Apply the reversion to free; every past plan, rating, recipe, pantry item, and the shared list stay fully available; the current week continues through Manual Weekly Planning | Standalone Automation, cross-feature | FEAT-14.SPEC-008 (FEAT-03, FEAT-05, FEAT-12, FEAT-23 responsibility for downstream behavior) |
| Downgrade/cancellation reversion completes | Send confirmation to Maya of what changed and what stayed | Standalone Notification | FEAT-14.SPEC-010 |
| Payment-processing capability reports a renewal payment failure (inbound event) | Set billing_state to Payment failed and start a 7-day grace timer; paid features remain active during grace | Standalone Automation | FEAT-14.SPEC-007 |
| Payment failure recorded | Send the organiser a grace-period notice with a path to update payment details | Standalone Notification, delivered through Standalone Integration | FEAT-14.SPEC-011 / FEAT-14.SPEC-012 |
| Maya updates payment details during the grace period | Retry the charge through the Payment Processing Integration | Standalone Integration | FEAT-14.SPEC-009 |
| Grace-period retry succeeds | Clear grace state, set billing_state back to Active, no gap in paid features | Standalone Automation | FEAT-14.SPEC-007 |
| Grace period (7 days) expires unresolved | Revert the household to free tier via Apply Subscription Change; no data is removed | Standalone Automation | FEAT-14.SPEC-007 (invokes FEAT-14.SPEC-008) |
| Grace-expiry reversion completes | Send confirmation to Maya that the household is now on the free tier | Standalone Notification | FEAT-14.SPEC-010 |
| Someone views or attempts to change billing | Show tier-only visibility (Sam, Riley) or full billing access (Maya) or deny entirely (unauthorized visitor, kid profiles), per role | Standalone Logic/Rule | FEAT-14.SPEC-006 |
| Maya views billing history | Load the billing_history list from the Payment Processing Integration | Inline in triggering screen | FEAT-14.SPEC-003 |

## Shared Context

**Shared Entities:**
- Subscription -- created by FEAT-01 (out of scope for this feature), read by SPEC-001 and SPEC-003, updated exclusively through SPEC-008 (called by SPEC-002, SPEC-003, SPEC-004, and SPEC-007). Fields: tier, billing_period, billing_state, billing_history.

**Shared UI Patterns:**
- Tier-inclusion summary -- the same "what each tier includes" content block appears on SPEC-001 (view), SPEC-002 (upgrade review), and SPEC-004 (what is kept vs. lost on downgrade). Spec Writers for all three screens should describe this content identically; only the surrounding action differs.
- Grace/billing-state banner -- a persistent, plain-language status indicator (e.g., "Payment failed — update your card by [date]") shown on SPEC-001 and SPEC-003 whenever billing_state is not Active; same wording source across both screens.

**Shared Validation/Logic:**
- FEAT-14.SPEC-005 defines the billing state and refund rules (valid payment, timing, grace length, no partial refund, organiser-only money flow); SPEC-002, SPEC-003, SPEC-004, and SPEC-008 all reference it rather than each re-deriving the timing or refund behavior.
- FEAT-14.SPEC-006 defines tier and billing access authorization; SPEC-001 and SPEC-003 both reference it rather than duplicating the per-role visibility logic.

## Internal Dependency Map

```
SPEC-001 (Plan Tier Overview) -> [Maya taps "Upgrade"] -> SPEC-002 (Upgrade to Paid)
SPEC-001 (Plan Tier Overview) -> [Maya taps "Manage Billing"] -> SPEC-003 (Billing & Payment Management)
SPEC-003 (Billing & Payment Management) -> [Maya taps "Downgrade" or "Cancel"] -> SPEC-004 (Downgrade / Cancel)
SPEC-002 -> [Maya submits payment] -> SPEC-009 (Payment Processing Integration) -> [charge succeeds] -> SPEC-008 (Apply Subscription Change) -> SPEC-001 [updated tier]
SPEC-003 -> [Maya switches billing period] -> SPEC-005 (Billing State & Refund Rules) [timing check] -> SPEC-008 (Apply Subscription Change) -> SPEC-001
SPEC-004 -> [Maya confirms downgrade or cancel] -> SPEC-005 (Billing State & Refund Rules) [period-end timing check] -> SPEC-008 (Apply Subscription Change) -> SPEC-001
SPEC-008 (Apply Subscription Change) -> [change applied] -> SPEC-010 (Billing Confirmation Notification) -> [delivered via] -> SPEC-012 (Transactional Email Delivery)
SPEC-009 (Payment Processing Integration) -> [inbound renewal-failure event] -> SPEC-007 (Payment Failure & Grace Period Handling)
SPEC-007 -> [grace notice needed] -> SPEC-011 (Payment Failure Grace-Period Notice) -> [delivered via] -> SPEC-012 (Transactional Email Delivery)
SPEC-007 -> [Maya updates payment during grace] -> SPEC-009 (retry charge) -> [success] -> SPEC-007 [clears grace state]
SPEC-007 -> [grace period expires unresolved] -> SPEC-008 (Apply Subscription Change) [reverts to free] -> SPEC-010
SPEC-001 -> [role-gated display] -> SPEC-006 (Tier & Billing Access Authorization)
SPEC-003 -> [role-gated access] -> SPEC-006 (Tier & Billing Access Authorization)
```

**Default Entry:** FEAT-14.SPEC-001 (Plan Tier Overview) -- the screen shown to any household member (per SPEC-006's authorization) who navigates to this feature area, whether directly, via FEAT-01's setup-complete "upgrade" choice, or via account settings; only Maya sees the Upgrade, Manage Billing, and Downgrade/Cancel entry points.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-14.SPEC-008 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Tier change gates AI plan generation; unlocked immediately on upgrade | Apply Subscription Change completes (XBR-05) |
| FEAT-14.SPEC-008 | Outbound | FEAT-05 (Pantry-Aware Suggestions) | Tier change gates the plan-weighting part of pantry-aware suggestions | Apply Subscription Change completes (XBR-05) |
| FEAT-14.SPEC-008 | Outbound | FEAT-12 (Meal Rating & Preference Learning) | Tier change gates the learning effect of ratings | Apply Subscription Change completes (XBR-05) |
| FEAT-14.SPEC-008 | Outbound | FEAT-23 (Manual Weekly Planning) | A downgraded or lapsed household continues planning through Manual Weekly Planning, with no data removed | Downgrade or grace-expiry reversion completes (XBR-05) |
| FEAT-14 (feature) | Inbound | FEAT-01 (Household Setup & Member Profiles) | Household created with Subscription defaulted to free; the setup-complete "upgrade" choice routes here | Setup complete, "upgrade" chosen (First Household Setup, step 6) |
| FEAT-14.SPEC-002 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Upgrade confirmation navigates to the first AI-generated plan on its way | Features unlock after subscribing (Upgrading & Managing the Account, step 3) |
| FEAT-14.SPEC-003 | Outbound | FEAT-18 (Account & Data Export) | Account settings navigation into a data-export request | Request an export (Upgrading & Managing the Account, step 4) |
| FEAT-14 (feature) | Inbound | FEAT-24 (Invite Another Household) | FEAT-24 reads whether a referred household's Subscription went on to pay, to attribute the referral (XBR-20) | Referral attribution check |
| FEAT-14.SPEC-001 | Inbound | FEAT-22 (Operator Read-Only Support Access) | Riley's read-only support access reads plan tier only, never payment details | Support views the household (Riley's Billing: View) |
| FEAT-14.SPEC-012 (dependency, resolved) | Outbound (dependency) | FEAT-01 (Household Setup & Member Profiles) | Billing confirmations and the grace-period notice are delivered through this feature's own transactional email boundary, distinct from FEAT-01.SPEC-017's account/recovery route, resolving the "pending" Transactional email row against FEAT-14 in feature-dependency-map.md's External Touchpoints | Every SPEC-010/SPEC-011 notification send |
| FEAT-14.SPEC-009 (dependency, resolved) | Outbound (dependency) | (none — capability owned entirely by this feature) | This Brief's Integration spec resolves the Payment processing row in feature-dependency-map.md's External Touchpoints, where FEAT-14 is listed as the only feature involved | Every upgrade, period switch, or grace-period retry |

## Non-Functional Notes

**Data volumes / growth:** Exactly one Subscription record per household, growing at the same rate as the household base — several thousand households in the first year (assumptions-constraints.md ASMP-24) — with billing_history retained for the life of the household account (scope-boundaries.md SC-18) rather than trimmed; free-tier households (the default for every new household) generate no AI cost tied to this feature (scope-boundaries.md SC-16).

**Responsiveness:** An upgrade action confirms within a few seconds (product-features.md, States); viewing the current tier works fully offline, while upgrading or changing billing requires connectivity (product-features.md, States: Offline-degraded). A failed upgrade attempt preserves the chosen plan option and offers a retry rather than losing the selection (product-features.md, States: Error).

**Data sensitivity / privacy:** Payment details are financial personal data, visible only to Maya and never to Sam, kids, or Riley — Riley's Billing access is View of the plan tier only, never payment details (feature-dependency-map.md, Subscription Data Sensitivity; user-persona.md Access Matrix notes). Payment details themselves are held with the payment-processing capability, not displayed or stored in the product beyond what billing history requires (feature-dependency-map.md, Subscription Relationships). General personal-data rights apply to this data — export and deletion — under assumptions-constraints.md ASMP-27.

**Compliance flags:** General personal-data protection (ASMP-27) covers billing and payment-related data, including the right to a copy and to deletion, exercised through FEAT-18; no medical-data regime applies (the product gives no medical or diet advice, scope-boundaries.md SC-06, unaffected by billing). The accessibility baseline (ASMP-29) applies directly to the Upgrade, Billing & Payment Management, and Downgrade/Cancel screens: one-thumb-reachable tap targets and plain-language billing-state messaging, never conveyed by color alone. Localization (ASMP-28) applies to billing amounts and history, shown in the household's configured currency (at least USD and GBP at launch).

## Non-Goals

- **Referral rewards, credits, or discounts affecting billing** -- Excluded per scope-boundaries.md SC-10: Invite Another Household records referrals without paying for them, keeping money flows limited strictly to the household subscription (product-features.md, Validation & Limits: "no money passes between households").
- **Partial refunds for unused subscription time** -- Excluded per product-features.md's Validation & Limits: cancelling keeps paid features until the end of the period already paid for, and no partial refunds are made for the unused remainder of a period.
- **Retroactively restricting a downgraded household's own history** -- Excluded per the Primary Flows & Alternates field's MODIFIED note, grounded in documented trust backlash from retroactive paywalling of a user's own history (Cozi, Trustpilot average 2.1/5, HIGH confidence): every past plan, rating, recipe, pantry item, and the shared list stay fully available after downgrade (XBR-05).
- **Billing for multiple households under one account** -- Excluded per scope-boundaries.md SC-03 ("one household per account in v1"): Subscription is a per-household entity with no multi-household billing consolidation.
- **Riley (Operator) ever viewing payment details** -- Excluded per the Access field and user-persona.md's Access Matrix notes: Riley's Billing access is View of the plan tier only, and never extends to payment details under any support scenario.
- **Automatic purge of billing history** -- Intentional lifecycle decision surfaced by the CRUD matrix: billing_history is retained for the life of the household account (scope-boundaries.md SC-18) with no automatic purge; it is removed only as part of the FEAT-18 account-deletion cascade.
- **A distinct native-app billing experience** -- Excluded per scope-boundaries.md SC-05: the product ships as a responsive web app for v1, with no native apps.



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



# Screen Spec: Upgrade to Paid

## Overview

**Name:** Upgrade to Paid
**ID:** FEAT-14.SPEC-002
**Type:** Screen
**Purpose:** Maya reviews what upgrading unlocks, chooses monthly or yearly billing, and enters payment details to subscribe.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Reviewing the tier-inclusion comparison in the context of upgrading
- Choosing monthly or yearly billing period
- Entering payment details and submitting the upgrade
- Preserving the chosen plan option and offering a retry when the upgrade attempt fails

**Non-Goals:**
- Viewing the current tier outside an upgrade attempt -- owned by FEAT-14.SPEC-001 (Plan Tier Overview), which is where this screen is entered from
- Updating payment details or viewing billing history after subscribing -- owned by FEAT-14.SPEC-003 (Billing & Payment Management)
- Submitting or retrying the actual charge -- owned by FEAT-14.SPEC-009 (Payment Processing Integration); this screen collects the input and displays the outcome
- Downgrading or cancelling -- excluded per this feature's own scope split: FEAT-14.SPEC-004 (Downgrade / Cancel) owns the opposite direction of this transition

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-14.SPEC-001 (Plan Tier Overview) | Maya taps "Upgrade" (shown only when tier is free) | None -- the form starts with no plan period preselected |
| FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) | Maya taps "Upgrade" on the free-tier plan placeholder | None -- the form starts with no plan period preselected |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | All actions: choose period, enter payment, submit | -- |
| Sam (Other Adult Member) | No | No | No entry point to this screen exists on FEAT-14.SPEC-001; a direct link resolves to "You don't have access to billing for this household." |
| Jordan (young kid profile, no login -- MVP) | No | No | Screen is unreachable -- a no-login profile has no sign-in path to any screen |
| Jordan (older kid, limited login -- Later) | No | No | A direct link resolves to "You don't have access to billing for this household." |
| Riley (Operator, support -- from v1) | No | No | Riley's Billing access is View of plan tier only, never payment collection; a direct link resolves to "You don't have access to billing for this household." |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-14.SPEC-001, not this form |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; the chosen plan period and any entered payment fields are preserved and restored after re-authentication (payment field values themselves are never persisted beyond the session, per FEAT-14.SPEC-009's Data Exchanged section -- only the period choice and non-sensitive form state survive) |

## Layout and Content

**Header:** Screen title "Upgrade to Paid" with a back arrow (returns to FEAT-14.SPEC-001) and no header action (Subscribe is in the footer).

**Body:**
- **Tier-inclusion summary block**, identical content to FEAT-14.SPEC-001 and FEAT-14.SPEC-004 per the Brief's Shared UI Patterns, framed here as "What you'll unlock."
- **Billing period selector**: two options, "Monthly" (platform parameter: `subscription-price-monthly` shown per the household's currency, per ASMP-28) and "Yearly" (platform parameter: `subscription-price-yearly`), presented as a toggle or radio pair; neither is preselected.
- **Payment details form**: standard payment-method fields collected on behalf of, and submitted through, FEAT-14.SPEC-009 (Payment Processing Integration) -- exact field set is that integration's contract, not redefined here.
- **Money-flow note**: a plain-language line stating that payment is between the organiser and the product; no money passes between households, per product-features.md's Validation & Limits.

**Footer:** "Subscribe" action button, full width, disabled until a billing period is chosen and the payment form is complete.

### Responsive Behavior

- **Compact breakpoint:** Tier-inclusion block, period selector, and payment form stack vertically, full width; "Subscribe" remains in the footer, one-thumb reachable per ASMP-29.
- **Medium size class and above:** Tier-inclusion block and period selector may render side by side above the payment form; no structural change to the form itself.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-14.SPEC-001 (Plan Tier Overview) | Screen closes | Standard transition; any entered payment data is discarded |
| "Monthly" option | Tap | Selects monthly billing period | Option shows selected state; price updates to the monthly amount | Selected-state styling |
| "Yearly" option | Tap | Selects yearly billing period | Option shows selected state; price updates to the yearly amount | Selected-state styling |
| Payment details fields | Type | Captures payment input per FEAT-14.SPEC-009's field contract | Field shows entered value | Standard input focus state |
| "Subscribe" button | Tap | 1. Validate a period is chosen and payment fields are complete, per FEAT-14.SPEC-005 (Billing State & Refund Rules) and FEAT-14.SPEC-009. 2. Submit the charge through FEAT-14.SPEC-009. 3. On success, trigger FEAT-14.SPEC-008 (Apply Subscription Change). | Button shows loading state during submission | Success: navigates to FEAT-14.SPEC-001 showing the new Paid tier, and FEAT-14.SPEC-010 (Billing Confirmation Notification) is sent. Failure: inline error per the Error state below; chosen period and non-sensitive form state are preserved |
| "Subscribe" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> tier-inclusion block -> "Monthly" option -> "Yearly" option -> payment details fields in order -> Subscribe.
- **Dynamic announcements:** A validation error on the payment form is announced to assistive technology and programmatically associated with the field; the submission-failure banner is announced as a live-region change.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | No period selected, payment fields empty, "Subscribe" disabled | Screen first opens | Maya selects a period or begins entering payment details |
| Filling | Period selected and/or payment fields contain input; "Subscribe" enabled once both are complete | Maya selects a period or types in a payment field | Maya taps Subscribe or navigates away |
| Submitting | "Subscribe" shows a loading spinner; a note reads "Confirming within a few seconds" per product-features.md's Loading state | Maya taps Subscribe with valid input | Submission completes (success or failure) |
| Error | Banner with the payment-processing capability's returned reason (per FEAT-14.SPEC-009's Degradation Behavior) and a Retry button; chosen period and non-sensitive form state remain | Submission fails (payment declined or processing error) | Maya taps Retry, or corrects input and resubmits |
| Offline/Degraded | Banner: "Upgrading requires a connection. Your choice is saved -- reconnect to finish." Period selection remains editable; payment fields remain visible but Subscribe is disabled | Connectivity is lost while this screen is open | Connectivity restored -- Subscribe re-enables; nothing is submitted automatically, since payment requires an explicit Subscribe tap even after reconnecting |

## Validation Rules

Validation governed by FEAT-14.SPEC-005 (Billing State & Refund Rules) for payment-validity requirements, and by FEAT-14.SPEC-009 (Payment Processing Integration) for payment field format rules. This screen applies validation on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-14.SPEC-001 (Plan Tier Overview) | -- |
| Successful subscribe | FEAT-14.SPEC-001 (Plan Tier Overview), showing the new Paid tier, with the first AI-generated plan on its way | FEAT-03.SPEC-004 (First Plan Generation on Upgrade) -- FEAT-03 |

## Data Model

**Creates:** None directly -- a successful submission triggers FEAT-14.SPEC-008 to write the Subscription update.
**Reads:** Subscription -- tier (to confirm the household is still free-tier on load). Household -- currency (to display prices in the household's configured currency).
**Updates:** None directly on this screen -- see FEAT-14.SPEC-008.
**Deletes:** None.

## Business Rules

- Valid payment details are required to upgrade, per FEAT-14.SPEC-005 -- Subscribe cannot succeed without them.
- A failed upgrade attempt never loses the chosen plan option: it stays selected and payment fields (except the sensitive payment values themselves, which follow FEAT-14.SPEC-009's own retention rules) are preserved for retry, per product-features.md's Error state.
- Successful submission triggers FEAT-14.SPEC-008 (Apply Subscription Change), which is the sole writer of the Subscription record -- this screen never writes tier or billing_period directly.
- XBR-05: on success, AI plan generation, pantry-aware plan weighting, and rating-based learning unlock immediately -- this screen's confirmation reflects that immediacy.

## Edge Cases

- **Payment declined or processing error** -- Inline in this screen per the Feature Breakdown Brief's Side-Effect Inventory: the chosen plan option is preserved, the payment form remains editable, and a Retry option is offered; no Subscription change occurs.
- **Maya double-taps Subscribe** -- The second tap is ignored while the first submission is in progress (button in loading state); at most one charge attempt is sent to FEAT-14.SPEC-009 per tap sequence.
- **Maya navigates away mid-submission** -- The submission continues; if it completes after she has left, the outcome is reflected the next time she opens FEAT-14.SPEC-001, and FEAT-14.SPEC-010 still sends its confirmation.
- **Household's tier changes to paid from another device while this screen is open (e.g., Maya subscribes on her phone while this screen is open on a laptop from an earlier session)** -- Tapping Subscribe here is rejected with "Your household is already on the paid tier." and the screen redirects to FEAT-14.SPEC-001, consistent with the dependency map's Contention note for Subscription (reject-with-refresh against a stale billing state).
- **Maya loses connectivity mid-entry, before tapping Subscribe** -- No data is lost; the Offline/Degraded state applies and Subscribe remains disabled until reconnection.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (inbound) | "Upgrade" hands off to this screen |
| FEAT-14.SPEC-005 (Billing State & Refund Rules) | References (inbound) | Payment-validity requirement enforced on submit |
| FEAT-14.SPEC-009 (Payment Processing Integration) | Triggers (outbound) | Submit action initiates the charge |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | Successful charge triggers the Subscription write |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Triggers (outbound) | Successful upgrade sends the confirmation |
| FEAT-03.SPEC-004 (First Plan Generation on Upgrade) | Navigation (outbound) | Successful upgrade routes toward the first AI-generated plan |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| upgrade_period_selected | period (monthly / yearly) | Maya selects a billing period | supports success-metrics.md: "Paid Conversion Rate" |
| upgrade_submitted | period, entry source | Maya taps Subscribe with valid input | supports success-metrics.md: "Paid Conversion Rate" |
| upgrade_completed | period | Payment succeeds and FEAT-14.SPEC-008 confirms tier=paid | supports success-metrics.md: "Paid Conversion Rate" |
| upgrade_failed | period, failure reason category | Payment is declined or a processing error occurs | supports success-metrics.md: "Paid Conversion Rate" (a failed attempt is a lost conversion this metric must reflect) |

## Acceptance Criteria

**FEAT-14.SPEC-002-AC-01:** Given Maya is on this screen, when she selects "Monthly", then the monthly price for her household's currency is shown and "Yearly" is deselected.

**FEAT-14.SPEC-002-AC-02:** Given Maya has selected "Yearly" and entered complete payment details, when she taps Subscribe, then the button shows a loading state and a note that confirmation takes a few seconds.

**FEAT-14.SPEC-002-AC-03:** Given Maya's payment succeeds, when the submission completes, then she is returned to FEAT-14.SPEC-001 showing "Paid -- Yearly", the confirmation notification (FEAT-14.SPEC-010) is sent, and she is offered the path toward her first AI-generated plan (FEAT-03.SPEC-004).

**FEAT-14.SPEC-002-AC-04:** Given Maya's payment is declined, when the submission fails, then her chosen period remains selected, the payment form remains editable, and a Retry option is shown -- no Subscription change occurs.

**FEAT-14.SPEC-002-AC-05:** Given Maya taps Subscribe with no billing period chosen, then the button remains disabled and no submission is attempted.

**FEAT-14.SPEC-002-AC-06:** Given Maya taps Subscribe twice in quick succession, when the first submission is still in progress, then the second tap has no effect and only one charge attempt is sent.

**FEAT-14.SPEC-002-AC-07:** Given Sam attempts to reach this screen directly, when the attempt is made, then he sees "You don't have access to billing for this household." and no form is shown.

**FEAT-14.SPEC-002-AC-08:** Given Maya loses connectivity while filling in payment details, when she taps Subscribe, then the banner "Upgrading requires a connection. Your choice is saved -- reconnect to finish." appears and Subscribe stays disabled until reconnection.

**FEAT-14.SPEC-002-AC-09:** Given Maya's household is upgraded to paid from another session while this screen remains open, when she taps Subscribe here, then the attempt is rejected with "Your household is already on the paid tier." and she is redirected to FEAT-14.SPEC-001.

**FEAT-14.SPEC-002-AC-10:** Given Maya navigates away while her submission is still processing, when the submission later succeeds, then the tier updates the next time she opens FEAT-14.SPEC-001, and the confirmation notification still arrives.

**FEAT-14.SPEC-002-AC-11:** Given an unauthenticated visitor opens a link to this screen, when the link resolves, then they are redirected to the sign-in screen and land on FEAT-14.SPEC-001 after signing in, not this form.

**FEAT-14.SPEC-002-AC-12:** Given Maya's session expires while she has entered a period choice and partial payment details, when she re-authenticates, then her period choice is restored to this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (empty, filling, submitting, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Billing & Payment Management

## Overview

**Name:** Billing & Payment Management
**ID:** FEAT-14.SPEC-003
**Type:** Screen
**Purpose:** Maya updates payment details, views billing history, and switches billing period for the household's paid subscription.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Updating payment details on the household's paid subscription
- Viewing billing history
- Switching billing period (monthly to yearly or yearly to monthly)
- Entry point into downgrading or cancelling
- Entry point into requesting a data export

**Non-Goals:**
- Upgrading from free to paid -- owned by FEAT-14.SPEC-002 (Upgrade to Paid); this screen is reachable only once the household is already paid
- The downgrade/cancel confirmation flow itself -- owned by FEAT-14.SPEC-004 (Downgrade / Cancel); this screen only offers the entry point
- Data export mechanics -- owned by FEAT-18 (Account & Data Management); this screen only offers the navigation entry point, per the Feature Breakdown Brief's Cross-Feature Touchpoints
- Determining timing for a period switch (next renewal, never mid-period) -- governed by FEAT-14.SPEC-005 (Billing State & Refund Rules); this screen collects the choice and defers to that spec's timing rule

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-14.SPEC-001 (Plan Tier Overview) | Maya taps "Manage Billing" (shown only when tier is paid) or the billing-state banner | None -- loads the household's current Subscription and billing_history |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Maya taps "View billing" | None -- loads the household's current Subscription and billing_history |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | Maya taps "Update payment details" | None -- loads the Subscription in its Payment failed billing_state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen, including payment details, billing history, and billing-state banner | All actions: update payment details, switch period, navigate to downgrade/cancel or export | -- |
| Sam (Other Adult Member) | No | No | No entry point to this screen exists on FEAT-14.SPEC-001; a direct link resolves to "You don't have access to billing for this household." |
| Jordan (young kid profile, no login -- MVP) | No | No | Screen is unreachable -- a no-login profile has no sign-in path to any screen |
| Jordan (older kid, limited login -- Later) | No | No | A direct link resolves to "You don't have access to billing for this household." |
| Riley (Operator, support -- from v1) | No | No | Riley's Billing access is View of plan tier only on FEAT-14.SPEC-001; payment details and billing history on this screen are never shown to Riley under any support scenario, per user-persona.md's Access Matrix notes |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-14.SPEC-001, not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; any in-progress payment-detail edit is preserved and restored after re-authentication (payment field values themselves follow FEAT-14.SPEC-009's retention rules) |

## Layout and Content

**Header:** Screen title "Manage Billing" with a back arrow (returns to FEAT-14.SPEC-001).

**Body:**
- **Billing-state banner**, at the top, shown when billing_state is not Active, or when pending_change is not none even while billing_state is Active (a pending downgrade); identical wording source to FEAT-14.SPEC-001 per the Brief's Shared UI Patterns.
- **Current plan summary**: billing_period ("Monthly" or "Yearly") with a "Switch to {other period}" action -- replaced, when pending_change is period_switch, by "Switching to {pending_change_new_period} on {current_period_end_date}" with no further switch action available; replaced, when pending_change is downgrade or billing_state is Cancelled, by "Your household moves to the free tier on {current_period_end_date}." with no switch action shown (there is no paid period left for a switch to apply to).
- **Payment details section**: the household's stored payment method summary (masked, per FEAT-14.SPEC-009's data-minimization contract) with an "Update payment details" action.
- **Billing history list**: chronological entries from billing_history, each showing date, amount, and outcome (charged, failed, refunded-status where applicable per FEAT-14.SPEC-005's no-partial-refund rule), sourced from FEAT-14.SPEC-009 (Payment Processing Integration).
- **Downgrade/Cancel entry**: a "Downgrade to Free" and a "Cancel Subscription" link, grouped together, visually separated from the sections above -- replaced, when pending_change is downgrade, by "Downgrade scheduled for {current_period_end_date}"; replaced, when billing_state is Cancelled, by "Cancellation scheduled for {current_period_end_date}"; in either case the remaining link (Cancel or Downgrade, respectively) is also hidden, since only one reversion to free can be pending at a time.
- **Data export entry**: a "Request a data export" link, per the Feature Breakdown Brief's Cross-Feature Touchpoints, navigating to FEAT-18.

**Footer:** None -- all actions are inline within their sections.

### Responsive Behavior

- **Compact breakpoint:** All sections stack vertically, full width; the billing history list scrolls independently within its section.
- **Medium size class and above:** Current plan summary and payment details sections may render side by side; the billing history list remains full width below them.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-14.SPEC-001 | Screen closes | Standard transition |
| Billing-state banner | Tap | No further navigation -- already on the billing screen it would otherwise lead to | None | -- |
| "Switch to {other period}" (shown only while pending_change is none) | Tap | 1. Confirm the switch via a dialog stating it takes effect at the next renewal, per FEAT-14.SPEC-005. 2. On confirm, trigger FEAT-14.SPEC-008 (Apply Subscription Change) to record the requested period (pending_change = period_switch). | Button shows a brief loading state | Success: toast "You'll switch to {period} billing at your next renewal." and the current plan summary updates to show the pending change. Failure: inline error with retry |
| "Update payment details" | Tap | Opens the payment-details form (fields per FEAT-14.SPEC-009's contract) | Form replaces the payment summary inline | Standard input focus state |
| Payment-details form Save | Tap | Submits updated payment details through FEAT-14.SPEC-009; while billing_state is Payment failed, the submission also triggers FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) to retry the charge | Button shows loading state | Success: toast "Payment details updated." and the masked summary reflects the new method. Failure: inline error, prior payment method remains in effect |
| Billing history list item | Tap | Display-only, non-interactive (no drill-down beyond the list row's own detail) | None | -- |
| "Downgrade to Free" | Tap | Navigate to FEAT-14.SPEC-004 (Downgrade / Cancel), downgrade path | Screen changes | Standard transition |
| "Cancel Subscription" | Tap | Navigate to FEAT-14.SPEC-004 (Downgrade / Cancel), cancellation path | Screen changes | Standard transition |
| "Request a data export" | Tap | Navigate to FEAT-18 (Account & Data Management) | Screen changes | Standard transition |

### Accessibility Notes

- **Focus order:** Back arrow -> billing-state banner (when shown) -> current plan summary and its switch action -> payment details section and its update action -> billing history list -> Downgrade/Cancel links -> data export link.
- **Dynamic announcements:** Toasts for period switch and payment-detail updates are announced as live-region changes; a validation error on the payment form is announced and associated with its field.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | All sections render with current data | Screen opens and Subscription and billing_history load successfully | User navigates away |
| Loading | Sections show loading placeholders | Screen first opens, before data has loaded | Load completes (success or error) |
| Error | Banner: "Couldn't load your billing details. Check your connection and try again." with Retry; no sections shown | Loading Subscription or billing_history fails | Retry succeeds |
| Empty (billing history) | Billing history section shows "No billing history yet" | The household has just upgraded and no charge has posted yet | The first charge posts and appears in the list |
| Offline/Degraded | Last-loaded plan summary, payment summary, and billing history remain visible with a banner: "You're offline -- showing your billing details as of your last visit." Switch period, update payment details, and downgrade/cancel actions are disabled with "Requires a connection." | Connectivity is lost while viewing, or the screen opens with cached data and no connectivity | Connectivity restored -- actions re-enable and data refreshes silently |

## Validation Rules

Validation governed by FEAT-14.SPEC-005 (Billing State & Refund Rules) for period-switch timing, and by FEAT-14.SPEC-009 (Payment Processing Integration) for payment field format rules. This screen applies validation on form submit for payment-detail updates and on confirmation for a period switch.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-14.SPEC-001 (Plan Tier Overview) | -- |
| "Downgrade to Free" tap | FEAT-14.SPEC-004 (Downgrade / Cancel) | -- |
| "Cancel Subscription" tap | FEAT-14.SPEC-004 (Downgrade / Cancel) | -- |
| "Request a data export" tap | FEAT-18 (Account & Data Management) | FEAT-18 |

## Data Model

**Creates:** None.
**Reads:** Subscription -- tier, billing_period, billing_state, billing_history, pending_change, pending_change_new_period, current_period_end_date (all displayed, including the scheduled-change and Cancelled-pending display). Household -- currency (billing history amounts displayed in the household's configured currency, per ASMP-28).
**Updates:** None directly -- period-switch and payment-detail changes are written by FEAT-14.SPEC-008 and FEAT-14.SPEC-009 respectively, triggered from this screen.
**Deletes:** None.

## Business Rules

- A period switch always takes effect at the next renewal, never immediately or mid-period, per FEAT-14.SPEC-005.
- Payment-detail updates take effect immediately for the next charge attempt; they do not themselves change tier, billing_period, or billing_state.
- Billing history is retained for the life of the household account with no purge, per scope-boundaries.md SC-18 -- this screen never offers a delete or clear action on history entries.
- Only Maya reaches this screen or any of its actions, per FEAT-14.SPEC-006 (Tier & Billing Access Authorization).
- The pending-change display (scheduled switch, scheduled downgrade, or scheduled cancellation) and the billing-state banner are both sourced from FEAT-14.SPEC-005's Governed Entity fields (pending_change, pending_change_new_period, current_period_end_date, billing_state) -- this screen never derives its own scheduled-change wording.

## Edge Cases

- **Subscription changed by a concurrent action (e.g., the grace period expires and the household reverts to free while this screen is open)** -- Any pending period-switch or payment-detail action is rejected with refresh: "Your billing state has changed. Reloading your billing details." and the screen reloads to reflect the current billing_state, consistent with the dependency map's Contention note for Subscription (reject-with-refresh against a stale billing state).
- **Maya taps "Switch to Yearly" twice in quick succession** -- The second tap is ignored while the first request is in progress; only one pending period-switch request exists at a time.
- **Payment-detail update fails validation** -- The prior payment method remains in effect and in use for the next charge attempt; no partial or invalid payment method is ever stored.
- **Maya requests a period switch, then downgrades before the next renewal** -- The downgrade (FEAT-14.SPEC-004, via FEAT-14.SPEC-008) supersedes the pending period switch; the household reverts to free at period end and the period-switch request never takes effect, since there is no paid period left for it to apply to.
- **Billing history contains an entry for a charge still processing** -- The entry shows a "Processing" outcome rather than a final charged/failed state until the outcome is known.
- **Maya opens this screen while a cancellation is already pending (billing_state Cancelled)** -- The Downgrade/Cancel section shows "Cancellation scheduled for {current_period_end_date}" in place of both links, and no "Switch to {other period}" action is shown, since the household is already resolving to free.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (inbound) | "Manage Billing" and the billing-state banner hand off to this screen |
| FEAT-14.SPEC-005 (Billing State & Refund Rules) | References (inbound) | Period-switch timing rule enforced here |
| FEAT-14.SPEC-009 (Payment Processing Integration) | Triggers (outbound) | Payment-detail updates and billing history are sourced from here |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | Period-switch requests are written here |
| FEAT-14.SPEC-004 (Downgrade / Cancel) | Navigation (outbound) | "Downgrade to Free" and "Cancel Subscription" lead here |
| FEAT-18 (Account & Data Management) | Navigation (outbound) | "Request a data export" hands off here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| billing_history_viewed | entry count | Screen finishes loading with billing_history populated | supports success-metrics.md: "Paying Household Retention" |
| period_switch_requested | from_period, to_period | Maya confirms a period switch | supports success-metrics.md: "Paying Household Retention" |
| payment_details_updated | outcome (success / failure) | Maya submits an updated payment method | supports success-metrics.md: "Paying Household Retention" |
| downgrade_or_cancel_entry_tapped | action (downgrade / cancel) | Maya taps "Downgrade to Free" or "Cancel Subscription" | N/A -- no Stage 2 metric tracks downgrade-intent taps directly; retained as a leading indicator ahead of "Paying Household Retention" |

## Acceptance Criteria

**FEAT-14.SPEC-003-AC-01:** Given Maya's household is on the paid monthly plan, when she opens this screen, then she sees "Monthly" as the current period with a "Switch to Yearly" action, her masked payment method, and her billing history.

**FEAT-14.SPEC-003-AC-02:** Given Maya taps "Switch to Yearly" and confirms, then a toast reads "You'll switch to Yearly billing at your next renewal." and the plan summary shows the pending change.

**FEAT-14.SPEC-003-AC-03:** Given Maya taps "Update payment details" and submits a new valid payment method, then a toast reads "Payment details updated." and the masked summary reflects it.

**FEAT-14.SPEC-003-AC-04:** Given Maya submits an invalid payment method, when validation fails, then the prior payment method remains in effect and an inline error is shown.

**FEAT-14.SPEC-003-AC-05:** Given Maya's household has billing_state Payment failed, when she opens this screen, then the billing-state banner appears at the top with the grace-period wording shared with FEAT-14.SPEC-001.

**FEAT-14.SPEC-003-AC-06:** Given Maya taps "Downgrade to Free", then she is navigated to FEAT-14.SPEC-004 on the downgrade path.

**FEAT-14.SPEC-003-AC-07:** Given Maya taps "Cancel Subscription", then she is navigated to FEAT-14.SPEC-004 on the cancellation path.

**FEAT-14.SPEC-003-AC-08:** Given Maya taps "Request a data export", then she is navigated to FEAT-18 (Account & Data Management).

**FEAT-14.SPEC-003-AC-09:** Given a household's grace period expires and it reverts to free while Maya has this screen open with a pending period-switch request, when the reversion completes, then her request is rejected with "Your billing state has changed. Reloading your billing details." and the screen reloads showing Free tier.

**FEAT-14.SPEC-003-AC-10:** Given Maya's household has just upgraded with no charges posted yet, when she opens this screen, then the billing history section shows "No billing history yet."

**FEAT-14.SPEC-003-AC-11:** Given Maya loses connectivity while viewing this screen, when she taps "Switch to Yearly", then the action is disabled with "Requires a connection." and her last-loaded billing details remain visible.

**FEAT-14.SPEC-003-AC-12:** Given Sam attempts to reach this screen directly, when the attempt is made, then he sees "You don't have access to billing for this household." and no billing data is shown.

**FEAT-14.SPEC-003-AC-13:** Given Maya's household has billing_state Cancelled with current_period_end_date March 14, when she opens this screen, then the Downgrade/Cancel section shows "Cancellation scheduled for March 14" instead of the Downgrade/Cancel links, and no "Switch to {other period}" action is shown.

**FEAT-14.SPEC-003-AC-14:** Given Maya's household has billing_state Active with pending_change downgrade and current_period_end_date March 14, when she opens this screen, then the current plan summary shows "Your household moves to the free tier on March 14." instead of a switch action.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 5 (loaded, loading, error, empty history, offline) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Downgrade / Cancel

## Overview

**Name:** Downgrade / Cancel
**ID:** FEAT-14.SPEC-004
**Type:** Screen
**Purpose:** Maya reviews exactly what is kept and what is lost, then confirms a downgrade to the free tier or a cancellation of the paid subscription.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Presenting the downgrade path and the cancellation path, each with an explanation of what is kept and what is lost
- Confirming a downgrade or cancellation request

**Non-Goals:**
- Executing the reversion to free at period end -- owned by FEAT-14.SPEC-008 (Apply Subscription Change), triggered by this screen's confirmation
- Determining the exact timing (end of current paid period, never mid-period) and the no-partial-refund rule -- governed by FEAT-14.SPEC-005 (Billing State & Refund Rules)
- Re-upgrading after a downgrade or cancellation -- owned by FEAT-14.SPEC-002 (Upgrade to Paid); this screen only ever moves the household toward free

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-14.SPEC-003 (Billing & Payment Management) | Maya taps "Downgrade to Free" | Path context: downgrade |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Maya taps "Cancel Subscription" | Path context: cancellation |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | All actions: review and confirm downgrade or cancellation | -- |
| Sam (Other Adult Member) | No | No | No entry point to this screen exists; a direct link resolves to "You don't have access to billing for this household." |
| Jordan (young kid profile, no login -- MVP) | No | No | Screen is unreachable -- a no-login profile has no sign-in path to any screen |
| Jordan (older kid, limited login -- Later) | No | No | A direct link resolves to "You don't have access to billing for this household." |
| Riley (Operator, support -- from v1) | No | No | Riley's Billing access never extends to this screen; unreachable under any support scenario |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-14.SPEC-001, not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; the chosen path (downgrade or cancellation) is preserved and this screen is restored after re-authentication, since no destructive action has occurred yet |

## Layout and Content

**Header:** Screen title "Downgrade to Free" or "Cancel Subscription" (matching the entered path) with a back arrow (returns to FEAT-14.SPEC-003).

**Body:**
- **What you keep / what you lose block**: identical content structure to the tier-inclusion summary on FEAT-14.SPEC-001 and FEAT-14.SPEC-002 per the Brief's Shared UI Patterns, framed for this screen as two labeled lists -- "You keep" (every past plan, rating, recipe, pantry item, and the shared list; manual weekly planning) and "You lose" (new AI-generated plans, pantry-aware plan weighting, and rating-based learning).
- **Timing explanation**: a plain-language statement that the change takes effect at the end of the current paid period (the exact date), and paid features remain active until then -- sourced from FEAT-14.SPEC-005.
- **No-refund note**: a plain-language line stating no partial refund is made for the unused remainder of the period, per FEAT-14.SPEC-005.
- **Confirmation control**: a single "Confirm downgrade" or "Confirm cancellation" button (matching the entered path).

**Footer:** None -- confirmation is inline in the body.

### Responsive Behavior

- **Compact breakpoint:** All blocks stack vertically, full width, one-thumb reachable per ASMP-29.
- **Medium size class and above:** "You keep" and "You lose" lists render side by side; the timing explanation, no-refund note, and confirmation control remain full width below.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-14.SPEC-003 (Billing & Payment Management) without confirming | Screen closes | Standard transition; no change is made |
| "What you keep / what you lose" block | -- | Display-only, non-interactive | None | -- |
| "Confirm downgrade" (downgrade path) | Tap | 1. Validate the household is currently paid with no change already pending, per FEAT-14.SPEC-005. 2. Trigger FEAT-14.SPEC-008 (Apply Subscription Change) to record the downgrade as pending against current_period_end_date; billing_state stays Active throughout. | Button shows loading state | Success: toast "Your household will move to the free tier on {current_period_end_date}. Paid features stay active until then." and navigation to FEAT-14.SPEC-001. Failure: inline error with retry |
| "Confirm cancellation" (cancellation path) | Tap | 1. Validate the household is currently paid with billing_state not already Cancelled, per FEAT-14.SPEC-005. 2. Trigger FEAT-14.SPEC-008 to set billing_state to Cancelled immediately and record the cancellation as pending against current_period_end_date. | Button shows loading state | Success: toast "Your subscription is cancelled. Paid features stay active until {current_period_end_date}, with no partial refund." and navigation to FEAT-14.SPEC-001. Failure: inline error with retry |

### Accessibility Notes

- **Focus order:** Back arrow -> "You keep" list -> "You lose" list -> timing explanation -> no-refund note -> confirmation button.
- **Dynamic announcements:** The success toast and any submission-failure banner are announced as live-region changes.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Reviewing (default) | Full explanation shown, confirmation button enabled | Screen opens from FEAT-14.SPEC-003 | Maya taps back or confirms |
| Confirming | Confirmation button shows a loading state | Maya taps "Confirm downgrade" or "Confirm cancellation" | Confirmation completes (success or failure) |
| Error | Banner: "Couldn't process your request. Check your connection and try again." with a Retry button; the review content remains unchanged | The confirmation request fails | Maya taps Retry and it succeeds |
| Offline/Degraded | Banner: "Changing billing requires a connection. Nothing has changed yet." Confirmation button is disabled | Connectivity is lost while this screen is open, or it is opened with no connectivity | Connectivity restored -- confirmation button re-enables |

## Validation Rules

Validation governed by FEAT-14.SPEC-005 (Billing State & Refund Rules). See that spec for the downgrade/cancellation timing rule and the condition that the household must currently be paid to reach a valid confirmation.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-14.SPEC-003 (Billing & Payment Management) | -- |
| Successful confirmation | FEAT-14.SPEC-001 (Plan Tier Overview) | -- |

## Data Model

**Creates:** None.
**Reads:** Subscription -- tier, billing_period, billing_state, pending_change, current_period_end_date (to confirm the household is currently paid, has no change already pending, and to compute the period-end date shown).
**Updates:** None directly -- the downgrade or cancellation is written by FEAT-14.SPEC-008, triggered from this screen.
**Deletes:** None.

## Business Rules

- A downgrade or cancellation always takes effect at the end of the current paid period, never mid-period, per FEAT-14.SPEC-005.
- A cancellation confirmation immediately sets billing_state to Cancelled (paid features remain active) and schedules the reversion to Reverted to free for current_period_end_date; a downgrade confirmation never changes billing_state during its pending window (it stays Active) and resolves to Active once tier reverts to free, per FEAT-14.SPEC-005 and FEAT-14.SPEC-008.
- No partial refund is made for the unused remainder of a period, per FEAT-14.SPEC-005 and product-features.md's Validation & Limits.
- XBR-05: every past plan, rating, recipe, pantry item, and the shared list stay fully available after the reversion completes -- nothing a free household already had is taken away. This is stated explicitly on this screen so the choice is fully informed, per the Non-Goals field's grounding in documented trust backlash from retroactive paywalling (Cozi, Trustpilot average 2.1/5, HIGH confidence).
- Only Maya reaches this screen, per FEAT-14.SPEC-006 (Tier & Billing Access Authorization).

## Edge Cases

- **Household's billing_state changes between load and confirmation (e.g., a grace-period reversion to free completes elsewhere while this screen is open)** -- Confirmation is rejected with refresh: "Your household is already on the free tier." and the screen redirects to FEAT-14.SPEC-001, consistent with the dependency map's Contention note for Subscription (reject-with-refresh against a stale billing state).
- **Maya taps Confirm twice in quick succession** -- The second tap is ignored while the first request is in progress; at most one downgrade or cancellation request is recorded per confirmation.
- **Maya backs out after reading the "what you lose" list** -- No change is made; tapping back returns her to FEAT-14.SPEC-003 with the subscription unchanged.
- **Maya cancels, then reconsiders before the period ends** -- Re-upgrading (FEAT-14.SPEC-002) before period end reverses the pending cancellation, since the household is still paid until then; this screen does not itself offer an "undo," but FEAT-14.SPEC-002 remains reachable throughout the remaining paid period.
- **Household has a pending period-switch request (from FEAT-14.SPEC-003) when Maya confirms a downgrade or cancellation here** -- The downgrade or cancellation supersedes the pending period switch; the switch never takes effect, since the household reverts to free before the next renewal it was scheduled for.
- **Maya reaches this screen via a stale link while a change is already pending for the household (e.g., billing_state is already Cancelled, or pending_change is already downgrade)** -- Confirmation is rejected with refresh: "Your household already has a change scheduled for {current_period_end_date}." and she is redirected to FEAT-14.SPEC-003, consistent with the dependency map's Contention note for Subscription.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-003 (Billing & Payment Management) | Navigation (inbound) | "Downgrade to Free" and "Cancel Subscription" hand off to this screen |
| FEAT-14.SPEC-005 (Billing State & Refund Rules) | References (inbound) | Timing and no-refund rules enforced here |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | Confirmation triggers the recorded downgrade or cancellation |
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (outbound) | Successful confirmation returns here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| downgrade_or_cancel_reviewed | path (downgrade / cancel) | Screen finishes loading | supports success-metrics.md: "Paying Household Retention" |
| downgrade_confirmed | -- | Maya confirms a downgrade | supports success-metrics.md: "Paying Household Retention" |
| cancellation_confirmed | -- | Maya confirms a cancellation | supports success-metrics.md: "Paying Household Retention" |
| downgrade_or_cancel_abandoned | path | Maya taps back without confirming | N/A -- no Stage 2 metric measures abandonment of this flow directly; retained as a diagnostic signal for retention efforts |

## Acceptance Criteria

**FEAT-14.SPEC-004-AC-01:** Given Maya arrives on the downgrade path, when the screen loads, then she sees "You keep" listing past plans, ratings, recipes, pantry items, and the shared list, and "You lose" listing AI-generated plans, pantry-aware weighting, and learning.

**FEAT-14.SPEC-004-AC-02:** Given Maya arrives on the cancellation path, when the screen loads, then the title reads "Cancel Subscription" and the same what-you-keep/what-you-lose content is shown with cancellation-specific confirmation wording.

**FEAT-14.SPEC-004-AC-03:** Given Maya is on the downgrade path, when she taps "Confirm downgrade", then a toast reads "Your household will move to the free tier on {period end date}. Paid features stay active until then." and she is returned to FEAT-14.SPEC-001.

**FEAT-14.SPEC-004-AC-04:** Given Maya is on the cancellation path, when she taps "Confirm cancellation", then a toast reads "Your subscription is cancelled. Paid features stay active until {period end date}, with no partial refund." and she is returned to FEAT-14.SPEC-001.

**FEAT-14.SPEC-004-AC-05:** Given Maya taps back on either path, when she has not confirmed, then no change is made to the Subscription and she returns to FEAT-14.SPEC-003.

**FEAT-14.SPEC-004-AC-06:** Given Maya's household reverts to free from a grace-period expiry while this screen is open, when she taps Confirm, then the request is rejected with "Your household is already on the free tier." and she is redirected to FEAT-14.SPEC-001.

**FEAT-14.SPEC-004-AC-07:** Given Maya taps Confirm twice in quick succession, when the first request is still in progress, then the second tap has no effect and only one request is recorded.

**FEAT-14.SPEC-004-AC-08:** Given Maya loses connectivity on this screen, when she taps Confirm, then the banner "Changing billing requires a connection. Nothing has changed yet." appears and the button is disabled.

**FEAT-14.SPEC-004-AC-09:** Given the confirmation request fails for a reason other than connectivity, when the failure occurs, then the banner "Couldn't process your request. Check your connection and try again." appears with a Retry button.

**FEAT-14.SPEC-004-AC-10:** Given Sam attempts to reach this screen directly, when the attempt is made, then he sees "You don't have access to billing for this household." and no content is shown.

**FEAT-14.SPEC-004-AC-11:** Given Maya's session expires while she is reviewing this screen, when she re-authenticates, then she is restored to this screen on the same path (downgrade or cancellation) she was reviewing.

**FEAT-14.SPEC-004-AC-12:** Given Maya's household already has a change pending (billing_state Cancelled, or pending_change downgrade) when she reaches this screen via a stale link, when she attempts to confirm, then the request is rejected with "Your household already has a change scheduled for {current_period_end_date}." and she is redirected to FEAT-14.SPEC-003.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 4 (reviewing, confirming, error, offline) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Billing State & Refund Rules

## Overview

**Name:** Billing State & Refund Rules
**ID:** FEAT-14.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs valid-payment requirements, downgrade/period-switch timing, the grace period, and the no-partial-refund and organiser-only money-flow constraints on the Subscription record.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management
**Governed Entity:** Subscription

## Scope and Non-Goals

**In Scope:**
- Valid-payment requirements for upgrading and for a grace-period retry
- Timing rules for downgrade, cancellation, and billing-period switches (always at period end / next renewal, never mid-period)
- The 7-day grace period following a renewal payment failure
- The no-partial-refund rule for the unused remainder of a period
- The organiser-only money-flow constraint

**Non-Goals:**
- Who may view or act on billing screens -- governed by FEAT-14.SPEC-006 (Tier & Billing Access Authorization); this spec governs the state and timing rules those screens must respect, not who reaches them
- Executing the Subscription write once a rule permits a change -- owned by FEAT-14.SPEC-008 (Apply Subscription Change); this spec defines what is allowed and when, not the write itself
- Detecting and reporting the payment failure or its retry outcome -- owned by FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) and FEAT-14.SPEC-009 (Payment Processing Integration); this spec defines the grace-period rule those specs enforce

## Governed Entity

**Entity:** Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| tier | enum (free, paid) | The household's current plan tier |
| billing_period | enum (none, monthly, yearly) | The household's billing cadence; none applies while tier is free |
| billing_state | enum (Active, Payment failed, Cancelled, Reverted to free) | The household's current billing status. Cancelled marks a confirmed cancellation that has not yet resolved -- set immediately on cancellation confirmation, paid features stay active, and it resolves to Reverted to free at current_period_end_date. Reverted to free is the terminal state for either a cancellation resolution or a grace-period lapse; a downgrade never passes through Cancelled and never resolves to Reverted to free -- it stays Active throughout and remains Active once tier reverts to free |
| billing_history | list | Chronological record of charges, outcomes, and amounts, visible to the organiser |
| current_period_end_date | date | The current paid period's end date -- equivalently the next renewal date while no change is pending, and the resolution date for any pending_change already scheduled against it. Null while tier is free |
| pending_change | enum (none, period_switch, downgrade, cancellation) | The single change, if any, scheduled to take effect at current_period_end_date. Distinct from billing_state: a pending downgrade or period switch leaves billing_state at Active, while a pending cancellation is the one case that also sets billing_state to Cancelled |
| pending_change_new_period | enum (none, monthly, yearly) | The incoming billing_period for a pending period switch; none unless pending_change is period_switch |
| payment_failure_date | date | The date of the most recent unresolved renewal payment failure; source for computing the grace-period end date (payment_failure_date + 7 days). Null unless billing_state is Payment failed |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-002 | Upgrade to Paid | On submit -- valid payment required before an upgrade can succeed |
| FEAT-14.SPEC-003 | Billing & Payment Management | On submit -- period-switch timing rule applied when Maya requests a switch |
| FEAT-14.SPEC-004 | Downgrade / Cancel | On submit -- downgrade/cancellation timing and no-partial-refund rule applied when Maya confirms |
| FEAT-14.SPEC-007 | Payment Failure & Grace Period Handling | During processing -- grace-period length and retry rules applied |
| FEAT-14.SPEC-008 | Apply Subscription Change | During processing -- every write to tier, billing_period, or billing_state validates against this spec's timing and state rules before committing |
| FEAT-14.SPEC-009 | Payment Processing Integration | On submit -- valid-payment-details format rules applied to charge and retry requests |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tier | Must be one of free, paid | Always | On every write | "Plan tier must be Free or Paid." | Yes |
| billing_period | Must be none while tier is free; must be monthly or yearly while tier is paid | Conditional on tier | On every write | "A billing period is required for a paid subscription." | Yes |
| billing_state | Must be one of Active, Payment failed, Cancelled, Reverted to free | Always | On every write | "Billing state must be a recognized value." | Yes |
| billing_state | Cannot be Payment failed or Cancelled while tier is free | Conditional on tier | On every write | "The free tier has no payment-failed or cancelled state." | Yes |
| billing_history | No validation beyond data type -- entries are appended by FEAT-14.SPEC-008 and FEAT-14.SPEC-009, never edited or removed by any role | Always | -- | -- | -- |
| current_period_end_date | Must be a valid date while tier is paid; must be null while tier is free | Conditional on tier | On every write | "A current period end date is required for a paid subscription." | Yes |
| pending_change | Must be one of none, period_switch, downgrade, cancellation; must be none while tier is free | Conditional on tier | On every write | "No change can be pending on a free-tier subscription." | Yes |
| pending_change_new_period | Must be monthly or yearly only when pending_change is period_switch; must be none for every other value of pending_change | Conditional on pending_change | On every write | "A new billing period only applies to a pending period switch." | Yes |
| payment_failure_date | Must be a valid date only when billing_state is Payment failed; must be null for every other billing_state | Conditional on billing_state | On every write | "A payment failure date only applies while billing_state is Payment failed." | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Valid payment required to upgrade | tier, billing_state | Upgrading from free to paid requires payment details that pass FEAT-14.SPEC-009's format and processing checks before tier can be set to paid | "Valid payment details are required to upgrade." |
| Downgrade is scheduled at period end without a billing_state change | tier, billing_period, billing_state, pending_change, current_period_end_date | A downgrade request never sets tier to free immediately while billing_state is Active on a paid tier; it sets pending_change to downgrade against the existing current_period_end_date, and billing_state stays Active throughout the pending window and remains Active once tier reverts to free at that date | "Downgrading takes effect at the end of your current billing period, not immediately." |
| Cancellation is recorded as Cancelled immediately and resolves at period end | tier, billing_period, billing_state, pending_change, current_period_end_date | A cancellation request sets billing_state to Cancelled immediately on confirmation (tier and billing_period stay unchanged and paid features remain active) and sets pending_change to cancellation against the existing current_period_end_date; when that date is reached, tier is set to free, billing_period is set to none, and billing_state is set to Reverted to free | "Cancelling takes effect at the end of your current billing period, not immediately." |
| Period switch takes effect at next renewal | billing_period, pending_change, pending_change_new_period, current_period_end_date | A billing_period change requested while tier is paid and billing_state is Active is recorded as pending_change = period_switch with pending_change_new_period set, applied only when current_period_end_date (the next renewal date) is reached, never mid-period | "Your billing period will change at your next renewal, not immediately." |
| Grace period is exactly 7 days | billing_state, payment_failure_date | A billing_state of Payment failed automatically reverts to Reverted to free if unresolved for 7 days from payment_failure_date, per FEAT-14.SPEC-007 | N/A -- system-timed transition, not a user-facing validation error |
| No partial refund | billing_state, billing_history | A downgrade, cancellation, or grace-expiry reversion never generates a refund entry in billing_history for the unused remainder of a period | "No refund is issued for the unused portion of your current billing period." |
| Organiser-only money flow | tier, billing_history | Every charge, retry, or refund-adjacent entry in billing_history is attributed to the household's own organiser; no money flow originates from or is directed to any other household (per XBR-20 and scope-boundaries.md SC-10) | "Billing applies only to your own household -- no money passes between households." |
| pending_change is exclusive | pending_change, pending_change_new_period | At most one of period_switch, downgrade, or cancellation can be pending for a Subscription at a time; confirming a downgrade or cancellation while a period_switch is already pending clears the period_switch (and its pending_change_new_period) entirely -- it never applies | "This replaces your previously requested billing-period change, which will no longer take effect." |

## Authorization Rules

{The base per-role question of who may view plan tier, view billing detail, or act on billing at all is governed by FEAT-14.SPEC-006 (Tier & Billing Access Authorization) -- this spec does not duplicate that table. The rows below cover only the billing-state and timing conditions layered on top of Maya's (Organiser's) own access, since those conditions are specific to this spec's rules rather than to role-based access.}

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Upgrade to paid | Maya (Organiser) | Only while tier is free (per FEAT-14.SPEC-006 for the base role check) | While tier is already paid: the attempt is rejected with "Your household is already on the paid tier." |
| Switch billing period | Maya (Organiser) | Only while tier is paid, billing_state is Active, and pending_change is none | While tier is free: no switch action shown. While billing_state is not Active: switch action disabled with "You can't switch billing periods while your billing state is {state}." While pending_change is downgrade or cancellation: no switch action shown, since the household is already resolving to free |
| Downgrade to free | Maya (Organiser) | Only while tier is paid and pending_change is not already downgrade or cancellation | While tier is free: no downgrade action shown. While pending_change is already downgrade or cancellation: the action is replaced by "Downgrade scheduled for {current_period_end_date}" / "Cancellation scheduled for {current_period_end_date}" with no further downgrade action to take |
| Cancel subscription | Maya (Organiser) | Only while tier is paid and billing_state is not already Cancelled | While tier is free: no cancel action shown. While billing_state is already Cancelled: the action is replaced by "Cancellation scheduled for {current_period_end_date}" with no further cancel action to take |
| Update payment details | Maya (Organiser) | Only while tier is paid | While tier is free: no payment-details section shown (no payment method exists to update) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| tier | free | On Household creation (FEAT-01.SPEC-011) | Yes -- Maya upgrades via FEAT-14.SPEC-002 |
| billing_period | none | On Household creation, while tier is free | Yes -- set to monthly or yearly on upgrade; switchable afterward |
| billing_state | Active | On Household creation, and restored on every successful upgrade or grace-period retry | No -- billing_state is always system-derived from payment and timing outcomes, never directly set by any role |
| billing_history | Empty list | On Household creation | No -- entries are appended only by FEAT-14.SPEC-008 and FEAT-14.SPEC-009 |
| current_period_end_date | Set to one billing_period ahead of the upgrade date on upgrade; advanced by one billing_period on every successful renewal or applied period switch; null on downgrade/cancellation/grace-expiry reversion to free | On upgrade, renewal, and period-switch application (FEAT-14.SPEC-008) | No -- system-derived, never directly set by any role |
| pending_change | none | On Household creation, and cleared back to none whenever FEAT-14.SPEC-008 applies the pending change at current_period_end_date | Set to period_switch, downgrade, or cancellation only via a confirmed request on FEAT-14.SPEC-003 or FEAT-14.SPEC-004 | No -- system-derived from a confirmed request, never directly set |
| pending_change_new_period | none | On Household creation, and cleared back to none whenever pending_change resolves or is superseded | Set only when pending_change is period_switch | No -- system-derived |
| payment_failure_date | null | On Household creation, and cleared on a successful upgrade or grace-period retry | Set by FEAT-14.SPEC-007 the moment a renewal payment failure is reported | No -- system-derived, never directly set by any role |

## Business Rules

- The payment grace period is 7 days from a renewal payment failure; if unresolved, the household reverts to free automatically, per FEAT-14.SPEC-007.
- Cancelling keeps paid features active until the end of the period already paid for; no partial refund is made for the unused remainder, per product-features.md's Validation & Limits.
- All money flows are between the organiser and the product through the payment-processing capability -- no money passes between households, per product-features.md's Validation & Limits and scope-boundaries.md SC-10.
- A downgrade or cancellation never retroactively restricts the household's own history -- every past plan, rating, recipe, pantry item, and the shared list stay fully available (XBR-05), consistent with scope-boundaries.md SC-18.
- Every Subscription write is routed exclusively through FEAT-14.SPEC-008 -- no screen or automation writes tier, billing_period, or billing_state directly, per the Feature Breakdown Brief's Shared Context.
- Cancellation and downgrade resolve to different terminal billing_state values: a cancellation is recorded as billing_state Cancelled immediately on confirmation and resolves to Reverted to free once current_period_end_date is reached, matching feature-overview.md's Entity-Lifecycle Coverage Matrix ("Active → Cancelled → Reverted to free at period end"). A downgrade never uses Cancelled -- it stays Active throughout its pending window and remains Active once tier reverts to free, since Reverted to free is reserved for a cancellation resolution or a grace-period lapse.
- current_period_end_date is the single date every pending_change resolves against; it is only ever set or advanced by FEAT-14.SPEC-008 on upgrade, renewal, or a period-switch application -- no screen edits it directly.

## Edge Cases

- **Downgrade requested on the exact last day of the current paid period** -- The reversion is scheduled for that period's end date like any other downgrade; if the period ends before the request is processed, the reversion applies at the literal boundary, not one day later.
- **Grace period reaches exactly 7 days with no resolution** -- The household reverts to free at the 7-day boundary; a resolution (successful retry) arriving in the same processing window as the 7-day boundary is honored if it completes before the reversion automation runs, per FEAT-14.SPEC-007's own ordering.
- **Period switch requested, then a downgrade confirmed before the next renewal** -- pending_change moves from period_switch to downgrade (pending_change_new_period is cleared); the switch never applies, since there is no paid period left for it to take effect on.
- **Cancellation confirmed while billing_state is already Cancelled (a stale confirmation resubmitted, e.g., from a second open tab)** -- The request is a no-op: billing_state stays Cancelled, pending_change remains cancellation against the same current_period_end_date, and no duplicate scheduling or billing_history entry is created.
- **Payment succeeds for an upgrade, but the household's tier was changed by a concurrent action moments earlier (e.g., Maya's household is already paid from another session)** -- The upgrade attempt here is rejected with "Your household is already on the paid tier."; no duplicate charge is recorded, since valid-payment-required applies only to a genuine free-to-paid transition.
- **A refund-adjacent entry is attempted for the unused remainder of a cancelled period** -- No such entry is ever created; the no-partial-refund rule blocks it at the rule level, not only at the confirmation-screen level, so no code path can bypass it by skipping FEAT-14.SPEC-004's screen.
- **Riley's support session ends (Support Request resolved) while viewing plan tier** -- Access is revoked at that moment; any further attempt to view returns "This support session has ended."

## Acceptance Criteria

**FEAT-14.SPEC-005-AC-01:** Given Maya's household is free tier, when she submits an upgrade with valid payment details, then the tier is permitted to change to paid.

**FEAT-14.SPEC-005-AC-02:** Given Maya's household is free tier, when she submits an upgrade with payment details that fail FEAT-14.SPEC-009's checks, then the upgrade is blocked with "Valid payment details are required to upgrade."

**FEAT-14.SPEC-005-AC-03:** Given Maya's household is paid with billing_state Active, when she confirms a downgrade, then the change is scheduled for the current period's end date, not applied immediately.

**FEAT-14.SPEC-005-AC-04:** Given Maya's household is paid, when she requests a billing-period switch, then the new period is recorded as pending and applies only at the next renewal date.

**FEAT-14.SPEC-005-AC-05:** Given a renewal payment fails, when billing_state is set to Payment failed, then a 7-day grace period begins during which paid features remain active.

**FEAT-14.SPEC-005-AC-06:** Given the grace period reaches 7 days with no successful retry, when the boundary is reached, then the household reverts to free tier automatically.

**FEAT-14.SPEC-005-AC-07:** Given Maya cancels her subscription, when the reversion to free occurs at period end, then no refund entry is created in billing_history for the unused remainder of the period.

**FEAT-14.SPEC-005-AC-08:** Given any billing_history entry is created, when it is examined, then it is attributed to the household's own organiser and never to or from another household.

**FEAT-14.SPEC-005-AC-09:** Given Maya's household is already paid, when she attempts to submit another upgrade, then it is rejected with "Your household is already on the paid tier."

**FEAT-14.SPEC-005-AC-10:** Given Maya's household is free tier, when she looks for a "Switch billing period" action, then none is shown, since billing_period only applies while tier is paid.

**FEAT-14.SPEC-005-AC-11:** Given Maya's household has billing_state Payment failed, when she attempts to switch billing period, then the action is disabled with "You can't switch billing periods while your billing state is Payment failed."

**FEAT-14.SPEC-005-AC-12:** Given a downgrade is requested exactly on the last day of the current paid period, when the request is confirmed, then the reversion is scheduled for that exact period-end date.

**FEAT-14.SPEC-005-AC-13:** Given Maya has a pending billing-period switch and then confirms a downgrade before the next renewal, when the downgrade completes, then the pending period switch never takes effect.

**FEAT-14.SPEC-005-AC-14:** Given every past plan, rating, recipe, pantry item, and the shared list existed before a downgrade, when the reversion to free completes, then all of them remain fully available and unrestricted.

**FEAT-14.SPEC-005-AC-15:** Given Maya's household is paid with billing_state Active, when she confirms a cancellation, then billing_state is set to Cancelled immediately, tier and billing_period remain unchanged, paid features stay active, and pending_change is recorded as cancellation against current_period_end_date.

**FEAT-14.SPEC-005-AC-16:** Given Maya's household has billing_state Cancelled and current_period_end_date is reached, when the reversion resolves, then tier is set to free, billing_period is set to none, and billing_state is set to Reverted to free (never Active).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 9 | 9 |
| Cross-Field Rules | 8 | 8 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 8 | 8 |
| Business Rules | 7 | 7 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Tier & Billing Access Authorization

## Overview

**Name:** Tier & Billing Access Authorization
**ID:** FEAT-14.SPEC-006
**Type:** Logic/Rule
**Purpose:** Enforces who can view tier status, who can change billing, and what an unauthorized visitor sees, per the Access Matrix.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management
**Governed Entity:** Subscription

## Scope and Non-Goals

**In Scope:**
- The base per-role authorization for viewing plan tier
- The base per-role authorization for viewing billing history and payment details
- The base per-role authorization for acting on billing (upgrade, manage billing, downgrade/cancel) at all, independent of billing state
- The unauthenticated and expired-session experience for every FEAT-14 screen
- Riley's Support-Request-gated, tier-only view

**Non-Goals:**
- Billing-state-conditioned and timing-conditioned rules layered on top of Maya's own access (e.g., that a period switch is disabled while billing_state is not Active) -- owned by FEAT-14.SPEC-005 (Billing State & Refund Rules), which references this spec for the base role check
- Executing any Subscription write -- owned by FEAT-14.SPEC-008 (Apply Subscription Change); this spec only governs who may initiate a request that could lead to one
- Opening or closing Riley's Support Request itself -- owned by FEAT-22 (Operator Read-Only Support Access); this spec only governs what Riley may see of Subscription data while a request is open, per XBR-14

## Governed Entity

**Entity:** Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| tier | enum (free, paid) | The household's current plan tier |
| billing_period | enum (none, monthly, yearly) | The household's billing cadence |
| billing_state | enum (Active, Payment failed, Cancelled, Reverted to free) | The household's current billing status |
| billing_history | list | Chronological record of charges, outcomes, and amounts |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-001 | Plan Tier Overview | On screen entry -- who sees the screen at all, and which sections and actions render |
| FEAT-14.SPEC-002 | Upgrade to Paid | On screen entry -- reachable only to Maya |
| FEAT-14.SPEC-003 | Billing & Payment Management | On screen entry -- reachable only to Maya |
| FEAT-14.SPEC-004 | Downgrade / Cancel | On screen entry -- reachable only to Maya |
| FEAT-14.SPEC-005 | Billing State & Refund Rules | References this spec for the base role check before layering its own billing-state conditions |
| FEAT-22 | Operator Read-Only Support Access | Governs when Riley's Support-Request-gated access opens or closes; this spec governs what that access may see of Subscription data while open |

## Field Validation Rules

{This spec governs access, not field content; every Subscription field's content rules are defined in FEAT-14.SPEC-005 (Billing State & Refund Rules). No validation beyond data type applies here.}

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tier | No validation beyond data type -- content rules owned by FEAT-14.SPEC-005 | Always | -- | -- | -- |
| billing_period | No validation beyond data type -- content rules owned by FEAT-14.SPEC-005 | Always | -- | -- | -- |
| billing_state | No validation beyond data type -- content rules owned by FEAT-14.SPEC-005 | Always | -- | -- | -- |
| billing_history | No validation beyond data type -- content rules owned by FEAT-14.SPEC-005 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Payment-detail fields never surface outside Maya's session | billing_history | Payment method summaries and billing_history entries are rendered only when the requesting role is Maya; every other role's request for this data is refused before any field value is read | "You don't have access to billing for this household." |
| Tier visibility is broader than billing-detail visibility | tier, billing_history | tier alone may be shown to Maya, Sam, and (while an open Support Request exists) Riley; billing_history and payment details are shown to Maya only | N/A -- this is a visibility scope rule, not a single error message; see Authorization Rules |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View plan tier | Maya (Organiser) | Always | -- |
| View plan tier | Sam (Other Adult Member) | Always -- every adult member can see which tier the household is on | -- |
| View plan tier | Jordan (young kid profile, no login -- MVP) | Never | Screen unreachable -- a no-login profile has no sign-in path to any screen |
| View plan tier | Jordan (older kid, limited login -- Later) | Never | No entry point is shown; a direct link resolves to "This isn't part of your household view." |
| View plan tier | Riley (Operator, support -- from v1) | Only while an open Support Request exists for the household (XBR-14) | Outside an open Support Request, the screen is unreachable to Riley; a session's access ends the moment the request is resolved, showing "This support session has ended." |
| View billing history and payment details | Maya (Organiser) | Always | -- |
| View billing history and payment details | Sam (Other Adult Member) | Never | "You don't have access to billing for this household." |
| View billing history and payment details | Both Jordan rows | Never | Screen unreachable (young kid); no entry point shown, direct link denied (older kid) |
| View billing history and payment details | Riley (Operator, support) | Never, under any support scenario | Not rendered even during an open Support Request -- Riley's Billing access is View of plan tier only, never payment details |
| Initiate any billing action (upgrade, manage billing, downgrade, cancel) | Maya (Organiser) | Always (subject to FEAT-14.SPEC-005's billing-state and timing conditions) | -- |
| Initiate any billing action | Sam, both Jordan rows, Riley | Never | "You don't have access to billing for this household." (Sam, older-kid row); screen unreachable (young kid, Riley) |
| Every FEAT-14 screen and action | Unauthenticated or expired-session visitor | Never | Redirected to the sign-in screen (unauthenticated); dialog "Your session has expired. Sign in to continue." (expired session) |

## Defaults and Derivations

{This spec governs access, not derived Subscription values; defaults and derivations for tier, billing_period, billing_state, and billing_history are owned by FEAT-14.SPEC-005.}

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Viewer's access level | Derived from the requesting role in the Access Matrix (user-persona.md) plus, for Riley, whether an open Support Request exists for the household | On every screen entry and every action attempt | No -- access level is never user-settable; it follows the product's role and support-request state |

## Business Rules

- Every adult member (Maya, Sam) can see which tier the household is on, but only Maya can act on billing or see billing history and payment details, per user-persona.md's Access Matrix.
- Riley's Billing access is View of the plan tier only and never extends to payment details under any support scenario, per user-persona.md's Access Matrix notes -- this is an absolute exclusion, not conditional on the reason for the support visit.
- Riley's access opens only against an open Support Request and closes the moment that request resolves, per XBR-14; every visit is recorded where Maya (Support View) can see it, per FEAT-22.
- Neither kid row (young or older) ever reaches any FEAT-14 screen or action -- the Access Matrix sets Billing to None for both rows unconditionally.
- No FEAT-14 screen conditionally reveals Maya-only controls to another role based on entity state (e.g., a "temporary" billing action for Sam) -- the role boundary is unconditional and independent of tier or billing_state.

## Edge Cases

- **Sam attempts to reach a Maya-only billing screen via a direct link while correctly signed in as himself** -- The screen is unreachable; the exact denied experience matches the screen's own Access and Visibility table (e.g., "You don't have access to billing for this household." on FEAT-14.SPEC-002, FEAT-14.SPEC-003, FEAT-14.SPEC-004).
- **Maya hands over the organiser role to Sam mid-session (FEAT-09 organiser hand-over) while Sam has this screen's tier-only view open** -- The moment the hand-over completes, Sam's access upgrades to full Billing access on his next screen load; his in-progress tier-only view does not retroactively grant him mid-session Maya-only actions without a reload.
- **Riley's Support Request resolves while Riley has the tier-only view open** -- Access is revoked immediately; any further action attempt returns "This support session has ended.", consistent with XBR-14's requirement that support access closes when the request is resolved.
- **An older-kid limited login (Later) is added to a household that later upgrades to paid** -- The older-kid row's Billing access remains None regardless of the household's tier; upgrading the household never changes any role's access level.
- **Unauthenticated visitor follows a deep link to billing history while household referral or invitation flows are also active** -- The visitor is redirected to sign-in exactly as for any other unauthenticated attempt; no referral or invitation context grants billing visibility.

## Acceptance Criteria

**FEAT-14.SPEC-006-AC-01:** Given Maya (Organiser) requests to view plan tier, when the request is made, then it is always allowed.

**FEAT-14.SPEC-006-AC-02:** Given Sam (Other Adult Member) requests to view plan tier, when the request is made, then it is allowed -- he sees the tier but no billing actions.

**FEAT-14.SPEC-006-AC-03:** Given Sam attempts to view billing history or payment details, when the attempt is made, then it is denied with "You don't have access to billing for this household."

**FEAT-14.SPEC-006-AC-04:** Given the young kid profile (no login, MVP) attempts to reach any FEAT-14 screen, when the attempt is made, then the screen is unreachable, since no sign-in path exists.

**FEAT-14.SPEC-006-AC-05:** Given the older-kid limited login (Later) attempts to reach any FEAT-14 screen, when the attempt is made, then no entry point is shown and a direct link resolves to "This isn't part of your household view."

**FEAT-14.SPEC-006-AC-06:** Given Riley has no open Support Request for a household, when Riley attempts to view its plan tier, then the screen is unreachable.

**FEAT-14.SPEC-006-AC-07:** Given Riley has an open Support Request for a household, when Riley views it, then only the plan tier is shown, never billing history or payment details.

**FEAT-14.SPEC-006-AC-08:** Given Riley's open Support Request resolves while Riley is viewing the tier, when the resolution completes, then Riley's access ends immediately and any further action shows "This support session has ended."

**FEAT-14.SPEC-006-AC-09:** Given Maya attempts to initiate an upgrade, manage billing, or a downgrade/cancellation, when the attempt is made, then it is allowed, subject to FEAT-14.SPEC-005's billing-state conditions.

**FEAT-14.SPEC-006-AC-10:** Given Sam attempts to initiate an upgrade, manage billing, or a downgrade/cancellation, when the attempt is made, then it is denied with "You don't have access to billing for this household."

**FEAT-14.SPEC-006-AC-11:** Given an unauthenticated visitor opens a link to any FEAT-14 screen, when the link resolves, then they are redirected to the sign-in screen.

**FEAT-14.SPEC-006-AC-12:** Given a signed-in member's session expires while on any FEAT-14 screen, when they attempt an action, then the dialog "Your session has expired. Sign in to continue." appears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Payment Failure & Grace Period Handling

## Overview

**Name:** Payment Failure & Grace Period Handling
**ID:** FEAT-14.SPEC-007
**Type:** Automation
**Purpose:** On a renewal payment failure, opens a 7-day grace period without cutting off access, and reverts the household to free if it lapses unresolved.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Opening the 7-day grace period the moment a renewal payment failure is reported
- Keeping paid features active throughout the grace period
- Processing a payment-details update during the grace period as a retry attempt
- Clearing the grace state on a successful retry
- Reverting the household to free when the grace period expires unresolved

**Non-Goals:**
- Detecting or reporting the underlying payment failure or retry outcome itself -- owned by FEAT-14.SPEC-009 (Payment Processing Integration); this automation only reacts to the events that spec reports
- Writing the final tier/billing_state values to the Subscription record -- owned by FEAT-14.SPEC-008 (Apply Subscription Change), which this automation invokes for the grace-expiry reversion
- Sending the grace-period notice itself -- owned by FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice); this automation only triggers it
- Collecting the updated payment details -- owned by FEAT-14.SPEC-003 (Billing & Payment Management), which is where Maya enters them during the grace period

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Renewal payment failure reported | FEAT-14.SPEC-009 (Payment Processing Integration) | Fires when the payment-processing capability reports a failed renewal charge for a household currently paid with billing_state Active | Subscription reference, failure reason (plain-language category) |
| Payment details updated during grace | FEAT-14.SPEC-003 (Billing & Payment Management) | Fires when Maya submits updated payment details while billing_state is Payment failed | Updated payment details (passed through to FEAT-14.SPEC-009 for the retry charge) |
| Grace-period boundary reached | System (schedule-based) | Fires once per Subscription, 7 days after payment_failure_date, only if billing_state is still Payment failed at that moment | Subscription reference, payment_failure_date |

## Processing Logic

1. Receive the renewal-payment-failure event from FEAT-14.SPEC-009 for a household currently on billing_state Active.
2. Set billing_state to Payment failed and record payment_failure_date as the failure date, starting the 7-day grace-period timer (grace period end = payment_failure_date + 7 days).
3. Confirm paid features (AI plan generation, pantry-aware plan weighting, rating-based learning) remain fully active -- no gating change accompanies this transition.
4. Trigger FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) to inform Maya.
5. If Maya updates payment details during the grace period (via FEAT-14.SPEC-003), submit a retry charge through FEAT-14.SPEC-009.
6. On a successful retry: clear the grace state (billing_state back to Active, payment_failure_date cleared), and record the successful charge in billing_history. Trigger FEAT-14.SPEC-010 (Billing Confirmation Notification) to confirm resolution.
7. On a failed retry: billing_state remains Payment failed and the grace-period timer is unaffected -- Maya may retry again at any point before the 7-day boundary.
8. If the 7-day boundary is reached with billing_state still Payment failed, invoke FEAT-14.SPEC-008 (Apply Subscription Change) to revert the household to free tier, setting billing_state to Reverted to free.
9. Confirm no data is removed by the reversion -- past plans, ratings, recipes, pantry items, and the shared list remain fully available, per XBR-05.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Grace period opened | Renewal payment failure reported for a paid, Active household | billing_state set to Payment failed; payment_failure_date recorded | Grace-period notice sent (FEAT-14.SPEC-011); billing-state banner appears on FEAT-14.SPEC-001 and FEAT-14.SPEC-003 | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-011 |
| Grace-period retry succeeds | Maya updates payment details during the grace period and the retry charge succeeds | billing_state set back to Active; payment_failure_date cleared; successful charge recorded in billing_history | Billing confirmation notification sent (FEAT-14.SPEC-010); billing-state banner disappears | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-010 |
| Grace-period retry fails | Maya updates payment details during the grace period but the retry charge is declined or errors | billing_state remains Payment failed; payment_failure_date unchanged; failed retry recorded in billing_history | FEAT-14.SPEC-003 shows the retry failure inline; the grace-period notice's remaining days are unaffected | FEAT-14.SPEC-003 |
| Grace period expires unresolved | 7 days elapse with billing_state still Payment failed | Household reverts to free via FEAT-14.SPEC-008: tier set to free, billing_period set to none, billing_state set to Reverted to free, payment_failure_date cleared | Confirmation of the reversion sent (FEAT-14.SPEC-010); no data removed | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-008, FEAT-14.SPEC-010 |
| Automation failure (grace-state transition itself fails to record) | An internal processing error prevents billing_state from being set on a reported failure | Household remains at its last-known billing_state (no partial change) | No user-visible failure message -- the payment-processing capability's own retry/backoff, per FEAT-14.SPEC-009, ensures the failure event is eventually processed | FEAT-14.SPEC-009 |

## Data Model

**Reads:** Subscription -- tier, billing_state, payment_failure_date (to confirm the household is eligible for grace handling at each step and to evaluate the 7-day boundary).
**Creates:** None.
**Updates:** Subscription -- billing_state (Active -> Payment failed -> Active or Reverted to free); payment_failure_date (set on failure, cleared on resolution); billing_history (appends the failure, retry, and resolution entries).
**Deletes:** None.

## Business Rules

- The payment grace period is exactly 7 days from the failure date; paid features remain fully active throughout, per product-features.md's Validation & Limits.
- A grace-period reversion is functionally identical to a downgrade: no data is removed, and every past plan, rating, recipe, pantry item, and the shared list stay fully available (XBR-05).
- The grace-period timer is not reset or extended by a failed retry attempt -- only a successful retry clears it; only the original 7-day boundary determines expiry.
- This automation is the sole trigger of a grace-period-driven reversion; every other tier or billing_state write is triggered elsewhere (upgrade, manual downgrade, cancellation) and is out of scope here.

## Edge Cases

- **Maya's retry succeeds on day 7 itself, at effectively the same moment the grace-expiry boundary check runs** -- The successful retry, once confirmed, takes precedence: if the retry's confirmation is recorded before the expiry check executes, billing_state is set to Active and no reversion occurs. If the expiry check executes first, the reversion proceeds and Maya's subsequent successful charge (now against a free-tier household) is treated as a fresh upgrade rather than a grace-period retry.
- **Maya submits payment details twice in quick succession during the grace period** -- Only one retry charge is in flight at a time; a second submission while the first retry is processing is held until the first resolves, preventing two concurrent charge attempts against the same Subscription.
- **The renewal-payment-failure event is delivered twice for the same failure** -- The second delivery changes nothing: billing_state is already Payment failed with the same failure date, and no duplicate grace-period notice is sent (per FEAT-14.SPEC-011's deduplication rule).
- **Concurrent trigger firing (a renewal-failure event and a grace-expiry boundary check for the same Subscription land at effectively the same time)** -- These cannot genuinely be concurrent: the grace-expiry check only fires 7 days after a failure was recorded, so a fresh failure event and an expiry check for a prior failure reference different failure dates and are processed independently, in event-time order.
- **Trigger fires while a previous run is in flight (e.g., a retry is processing when the expiry boundary is reached)** -- The expiry check waits for the in-flight retry to resolve before evaluating billing_state, so a retry that resolves at the boundary is never overtaken mid-processing by the reversion.
- **Household is deleted (FEAT-18 cascade) while in a grace period** -- The grace-period timer and any pending retry are cancelled silently along with the rest of the household's data; no reversion or notice fires for a deleted household.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-009 (Payment Processing Integration) | Triggered by (inbound) | The renewal-payment-failure event fires this automation |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Triggered by (inbound) | Maya's payment-details update during grace fires the retry path |
| FEAT-14.SPEC-009 (Payment Processing Integration) | Triggers (outbound) | Retry charges are submitted through this integration |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | Triggers (outbound) | Grace period opening fires this notification |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Triggers (outbound) | A successful retry or grace-expiry reversion fires this notification |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | Grace-expiry reversion is executed through this automation |
| FEAT-14.SPEC-001 (Plan Tier Overview) | Affects (outbound) | The billing-state banner reflects every state this automation sets |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Affects (outbound) | The billing-state banner and retry outcome are shown here |

## Analytics and Success Signals

- **subscription_payment_failed** (billing period) -- supports success-metrics.md: "Paying Household Retention"
- **subscription_grace_period_opened** (days remaining: 7) -- supports success-metrics.md: "Paying Household Retention"
- **subscription_grace_retry_succeeded** (days elapsed since failure) -- supports success-metrics.md: "Paying Household Retention"
- **subscription_grace_retry_failed** (days elapsed since failure) -- N/A -- no Stage 2 metric tracks individual retry failures; retained as an operational signal for grace-period recovery rate
- **subscription_grace_expired** (billing period at time of expiry) -- supports success-metrics.md: "Paying Household Retention" (a lapsed grace period is a retention loss this metric must reflect)

## Acceptance Criteria

**FEAT-14.SPEC-007-AC-01:** Given a paid household with billing_state Active, when FEAT-14.SPEC-009 reports a renewal payment failure, then billing_state is set to Payment failed, payment_failure_date is recorded, the 7-day grace timer starts, and paid features remain active.

**FEAT-14.SPEC-007-AC-02:** Given a household enters the grace period, when the transition completes, then FEAT-14.SPEC-011 sends Maya the grace-period notice.

**FEAT-14.SPEC-007-AC-03:** Given Maya is in a grace period and updates her payment details on FEAT-14.SPEC-003, when the retry charge succeeds, then billing_state is set back to Active and FEAT-14.SPEC-010 sends a confirmation.

**FEAT-14.SPEC-007-AC-04:** Given Maya is in a grace period and updates her payment details, when the retry charge fails, then billing_state remains Payment failed and the grace-period timer is unaffected.

**FEAT-14.SPEC-007-AC-05:** Given a household has been in the grace period for 7 days with no successful retry, when the expiry boundary is reached, then the household reverts to free tier via FEAT-14.SPEC-008 and FEAT-14.SPEC-010 sends a confirmation.

**FEAT-14.SPEC-007-AC-06:** Given a household's grace-period reversion completes, when Maya checks her data, then every past plan, rating, recipe, pantry item, and the shared list remain fully available.

**FEAT-14.SPEC-007-AC-07:** Given Maya's retry succeeds at effectively the same moment the 7-day expiry boundary is reached, when the retry's confirmation is recorded first, then billing_state is set to Active and no reversion occurs.

**FEAT-14.SPEC-007-AC-08:** Given Maya submits payment details twice in quick succession during a grace period, when the first retry is still processing, then the second submission is held until the first resolves, and no two retry charges are ever in flight at once.

**FEAT-14.SPEC-007-AC-09:** Given the same renewal-payment-failure event is delivered twice, when the second delivery arrives, then billing_state is unchanged and no duplicate grace-period notice is sent.

**FEAT-14.SPEC-007-AC-10:** Given a retry is processing when the 7-day expiry boundary is reached, when the expiry check runs, then it waits for the in-flight retry to resolve before evaluating billing_state.

**FEAT-14.SPEC-007-AC-11:** Given a household is deleted while in a grace period, when the deletion completes, then the pending grace-period timer and any in-flight retry are cancelled silently with no reversion or notice.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (failure reported, payment updated during grace, grace-boundary reached) | 3 |
| Outcome Paths | 5 (opened, retry succeeds, retry fails, expires, automation failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Apply Subscription Change

## Overview

**Name:** Apply Subscription Change
**ID:** FEAT-14.SPEC-008
**Type:** Automation
**Purpose:** Writes every tier, billing-period, and billing-state change to the Subscription record and signals the features that gate on it.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- The single, shared write path for every Subscription change: upgrade, billing-period switch, downgrade, cancellation, and grace-expiry reversion
- Signaling FEAT-03, FEAT-05, and FEAT-12 that tier gating has changed
- Signaling FEAT-23 that a household now plans manually
- Triggering the billing confirmation notification for every change it applies

**Non-Goals:**
- Deciding whether a requested change is currently allowed (payment validity, timing, grace-period state) -- owned by FEAT-14.SPEC-005 (Billing State & Refund Rules); this automation applies changes that have already passed that spec's rules
- Deciding who may request a change -- owned by FEAT-14.SPEC-006 (Tier & Billing Access Authorization)
- Collecting payment details or submitting a charge -- owned by FEAT-14.SPEC-009 (Payment Processing Integration); this automation is invoked only after a charge (where one is needed) has already succeeded
- Detecting a payment failure or grace-period expiry itself -- owned by FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling), which invokes this automation for the grace-expiry reversion

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Upgrade payment succeeds | FEAT-14.SPEC-002 (Upgrade to Paid) via FEAT-14.SPEC-009 (Payment Processing Integration) | Fires when a charge for a free-to-paid upgrade succeeds | Household reference, chosen billing_period (monthly / yearly) |
| Billing-period switch requested | FEAT-14.SPEC-003 (Billing & Payment Management) | Fires when Maya confirms a period switch, per FEAT-14.SPEC-005's timing rule | Household reference, requested billing_period, effective date (next renewal) |
| Downgrade confirmed | FEAT-14.SPEC-004 (Downgrade / Cancel), downgrade path | Fires when Maya confirms a downgrade, per FEAT-14.SPEC-005's timing rule; only while no change is already pending | Household reference, current_period_end_date (the resolution date) |
| Cancellation confirmed | FEAT-14.SPEC-004 (Downgrade / Cancel), cancellation path | Fires when Maya confirms a cancellation, per FEAT-14.SPEC-005's timing rule; only while billing_state is not already Cancelled | Household reference, current_period_end_date (the resolution date) |
| Grace-period expired unresolved | FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Fires when the 7-day grace period lapses with no successful retry | Household reference |

## Processing Logic

1. Receive the requested change and its effective timing from the triggering spec.
2. For an immediately-effective change with no pending window (upgrade, grace-expiry reversion): write the new tier, billing_period, and billing_state to the Subscription record at once. On an upgrade, also set current_period_end_date to one billing_period ahead and clear pending_change/pending_change_new_period/payment_failure_date.
3. For a cancellation confirmation: write billing_state to Cancelled immediately -- tier and billing_period stay unchanged and paid features remain active -- and set pending_change to cancellation against the household's existing current_period_end_date. This does not wait for period end; only the tier/billing_period reversion does (step 5).
4. For a period-end-effective change with no immediate state write (downgrade, a confirmed period switch): record pending_change (downgrade, or period_switch with pending_change_new_period set) against the existing current_period_end_date; if a different pending_change was already set (e.g., a period switch), it is replaced and its pending_change_new_period cleared -- at most one pending_change exists at a time. The current tier, billing_period, and billing_state remain unchanged until current_period_end_date is reached.
5. When current_period_end_date is reached for a Subscription with a pending_change: if pending_change is downgrade, set tier to free, billing_period to none, billing_state to Active, clear current_period_end_date, and clear pending_change. If pending_change is cancellation, set tier to free, billing_period to none, billing_state to Reverted to free, clear current_period_end_date, and clear pending_change. If pending_change is period_switch, update billing_period to pending_change_new_period, advance current_period_end_date by the new billing_period, and clear pending_change and pending_change_new_period.
6. Append an entry to billing_history recording the change type, the amount involved (where applicable, per platform parameter: `subscription-price-monthly` or platform parameter: `subscription-price-yearly`), and the date applied. A cancellation's immediate Cancelled recording (step 3) does not itself append a billing_history entry -- only its eventual resolution (step 5) does, since no charge or refund occurs at confirmation time.
7. Once tier changes, signal FEAT-03 (AI Weekly Dinner Plan Generation), FEAT-05 (Pantry-Aware Suggestions), and FEAT-12 (Meal Rating & Preference Learning) so each re-evaluates its own tier gating on its next relevant action (XBR-05).
8. When tier moves to free (downgrade, cancellation, or grace-expiry reversion), signal FEAT-23 (Manual Weekly Planning) so the household's current and future weeks route there.
9. Confirm no plan, rating, recipe, pantry item, or list data is altered or removed by any tier change -- this automation only ever writes to the Subscription record itself.
10. Trigger FEAT-14.SPEC-010 (Billing Confirmation Notification) with the applied change's details. The immediate Cancelled recording (step 3) does not itself trigger this notification -- FEAT-14.SPEC-004's own confirmation toast covers that moment; FEAT-14.SPEC-010 fires only once a change actually resolves (upgrade, period-switch application, downgrade/cancellation reversion, or grace-expiry reversion).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Upgrade applied | Upgrade payment succeeded | tier set to paid; billing_period set to the chosen period; billing_state set to Active; current_period_end_date set one billing_period ahead; billing_history entry appended | FEAT-14.SPEC-001 shows the new Paid tier; FEAT-14.SPEC-010 sends the upgrade confirmation | FEAT-14.SPEC-001, FEAT-14.SPEC-002, FEAT-14.SPEC-010, FEAT-03 (FEAT-03.SPEC-004), FEAT-05, FEAT-12, FEAT-24.SPEC-005 |
| Period switch scheduled | Maya confirms a period switch | pending_change set to period_switch; pending_change_new_period set to the chosen period; billing_period unchanged until current_period_end_date | FEAT-14.SPEC-003 shows the pending switch | FEAT-14.SPEC-003 |
| Period switch applied | current_period_end_date is reached with pending_change period_switch | billing_period updated to pending_change_new_period; current_period_end_date advanced by the new period; pending_change and pending_change_new_period cleared; billing_history entry appended | FEAT-14.SPEC-010 sends the period-switch confirmation | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-010 |
| Downgrade scheduled | Maya confirms a downgrade | pending_change set to downgrade against the existing current_period_end_date; tier, billing_period, billing_state unchanged (billing_state stays Active) | FEAT-14.SPEC-001 and FEAT-14.SPEC-003 show the scheduled reversion date | FEAT-14.SPEC-001, FEAT-14.SPEC-003 |
| Downgrade applied | current_period_end_date is reached with pending_change downgrade | tier set to free; billing_period set to none; billing_state set to Active; current_period_end_date and pending_change cleared; billing_history entry appended | FEAT-14.SPEC-010 sends the reversion confirmation; FEAT-23 becomes the household's planning route | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-010, FEAT-23 |
| Cancellation recorded | Maya confirms a cancellation | billing_state set to Cancelled immediately; pending_change set to cancellation against the existing current_period_end_date; tier and billing_period unchanged; paid features remain active; no billing_history entry (no charge or refund occurs yet) | FEAT-14.SPEC-001 shows the Cancelled banner; FEAT-14.SPEC-003 shows "Cancellation scheduled for {current_period_end_date}" in place of the Downgrade/Cancel links | FEAT-14.SPEC-001, FEAT-14.SPEC-003 |
| Cancellation applied | current_period_end_date is reached with billing_state Cancelled and pending_change cancellation | tier set to free; billing_period set to none; billing_state set to Reverted to free (never Active); current_period_end_date and pending_change cleared; billing_history entry appended | FEAT-14.SPEC-010 sends the reversion confirmation; FEAT-23 becomes the household's planning route | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-010, FEAT-23 |
| Grace-expiry reversion applied | FEAT-14.SPEC-007 invokes this automation after 7 unresolved days | tier set to free; billing_period set to none; billing_state set to Reverted to free; payment_failure_date cleared; billing_history entry appended | FEAT-14.SPEC-010 sends the reversion confirmation; FEAT-23 becomes the household's planning route | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-007, FEAT-14.SPEC-010, FEAT-23 |
| Automation failure (write cannot be committed) | An internal processing error prevents the Subscription write | No partial write -- the Subscription record retains its prior, fully consistent state | The triggering screen's own error state applies (e.g., FEAT-14.SPEC-002 shows its Error state); no gating signal is sent, since no change occurred | The triggering spec |

## Data Model

**Reads:** Subscription -- current tier, billing_period, billing_state, pending_change, pending_change_new_period, current_period_end_date (to confirm the write is still valid against the state it was requested against, per FEAT-14.SPEC-005's reject-with-refresh rule).
**Creates:** None.
**Updates:** Subscription -- tier, billing_period, billing_state, billing_history, current_period_end_date, pending_change, pending_change_new_period, payment_failure_date (the sole writer of these fields across the entire product, per the Feature Breakdown Brief's Shared Context).
**Deletes:** None.

## Business Rules

- This automation is the exclusive writer of the Subscription record's tier, billing_period, billing_state, current_period_end_date, pending_change, pending_change_new_period, and payment_failure_date -- FEAT-14.SPEC-002, SPEC-003, SPEC-004, and SPEC-007 all invoke it rather than writing directly, per the Feature Breakdown Brief's Internal Dependency Map.
- A cancellation is recorded as billing_state Cancelled immediately on confirmation and resolves to Reverted to free once current_period_end_date is reached, matching feature-overview.md's Entity-Lifecycle Coverage Matrix ("Active → Cancelled → Reverted to free at period end"). A downgrade, by contrast, never passes through Cancelled -- billing_state stays Active throughout its pending window and remains Active once tier reverts to free at period end.
- No plan, rating, recipe, pantry item, or list data is ever altered by a tier change, per XBR-05 -- this automation's writes are scoped strictly to the Subscription record.
- XBR-05: FEAT-03, FEAT-05, and FEAT-12's tier gating always reflects the current tier the moment this automation applies a change -- there is no delay between a completed change and gated features unlocking or locking.

## Edge Cases

- **Two pending changes exist at once (a scheduled period switch and a later-confirmed downgrade or cancellation)** -- The downgrade or cancellation supersedes the pending period switch entirely (pending_change and pending_change_new_period are overwritten); when current_period_end_date is reached, tier moves to free and the period switch never applies, since there is no paid period left for it to affect.
- **A cancellation is confirmed a second time while billing_state is already Cancelled (a stale confirmation resubmitted)** -- The request is a no-op: billing_state stays Cancelled, pending_change remains cancellation against the same current_period_end_date, and no duplicate billing_history entry or notification is produced.
- **The scheduled effective date for a pending downgrade, cancellation, or period switch is reached while the household is mid-grace-period (a payment failure occurred after the change was scheduled but before its effective date)** -- The grace-period reversion (FEAT-14.SPEC-007) and the pending scheduled change converge on the same free-tier outcome; whichever completes first sets tier to free and billing_state to Reverted to free, clearing pending_change, and the other is a no-op against an already-free household.
- **Concurrent trigger firing (an upgrade payment succeeds at effectively the same moment a stale downgrade confirmation from an earlier session also fires)** -- Reject-with-refresh applies, per the dependency map's Contention note for Subscription: the change that reads the current billing_state first proceeds; the second is refused and shown the current, just-updated state.
- **Trigger fires while a previous run is in flight (e.g., a period-switch application and a downgrade both reach their effective moment together)** -- Writes to the same Subscription record are serialized: the second trigger's processing waits for the first to complete, then re-reads the just-updated state before applying, so no write is lost or overwritten silently.
- **A signal to FEAT-03, FEAT-05, or FEAT-12 is not acknowledged (e.g., one of those features is momentarily unavailable)** -- The Subscription write itself has already committed; each gated feature re-evaluates tier from the Subscription record directly on its own next action, so a missed signal never leaves a feature permanently out of sync -- it only delays that feature noticing the change until its next read.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-002 (Upgrade to Paid) | Triggered by (inbound) | A successful upgrade payment fires this automation |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Triggered by (inbound) | A confirmed period switch fires this automation |
| FEAT-14.SPEC-004 (Downgrade / Cancel) | Triggered by (inbound) | A confirmed downgrade or cancellation fires this automation |
| FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Triggered by (inbound) | A grace-expiry reversion fires this automation |
| FEAT-14.SPEC-005 (Billing State & Refund Rules) | References (inbound) | Timing and validity rules this automation applies changes under |
| FEAT-14.SPEC-001 (Plan Tier Overview) | Affects (outbound) | Reflects every applied and pending change |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Affects (outbound) | Reflects every applied and pending change |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Triggers (outbound) | Every applied change fires this notification |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Affects (outbound) | Tier gating for AI plan generation updates |
| FEAT-05 (Pantry-Aware Suggestions) | Affects (outbound) | Tier gating for plan-weighting updates |
| FEAT-12 (Meal Rating & Preference Learning) | Affects (outbound) | Tier gating for the learning effect updates |
| FEAT-23 (Manual Weekly Planning) | Affects (outbound) | Becomes the household's planning route when tier moves to free |

## Analytics and Success Signals

- **subscription_upgraded** (billing_period) -- supports success-metrics.md: "Paid Conversion Rate"
- **subscription_period_switched** (from_period, to_period) -- supports success-metrics.md: "Paying Household Retention"
- **subscription_downgraded** (reason: organiser_initiated) -- supports success-metrics.md: "Paying Household Retention"
- **subscription_cancellation_scheduled** (reason: organiser_initiated) -- supports success-metrics.md: "Paying Household Retention" -- emitted the moment billing_state is set to Cancelled
- **subscription_cancelled** (reason: organiser_initiated) -- supports success-metrics.md: "Paying Household Retention" -- emitted when the Cancelled-to-Reverted-to-free resolution completes at current_period_end_date
- **subscription_grace_expired** (billing_period at time of expiry) -- supports success-metrics.md: "Paying Household Retention"

## Acceptance Criteria

**FEAT-14.SPEC-008-AC-01:** Given Maya's upgrade payment succeeds on FEAT-14.SPEC-002, when this automation fires, then tier is set to paid, billing_period is set to her chosen period, billing_state is set to Active, and FEAT-14.SPEC-010 sends the upgrade confirmation.

**FEAT-14.SPEC-008-AC-02:** Given the upgrade completes, when FEAT-03 next checks tier gating, then it correctly reads the household as paid and permits AI plan generation.

**FEAT-14.SPEC-008-AC-03:** Given Maya confirms a period switch from monthly to yearly, when this automation fires, then the switch is recorded as pending with a next-renewal effective date, and billing_period is unchanged until then.

**FEAT-14.SPEC-008-AC-04:** Given a pending period switch's renewal date is reached, when this automation applies it, then billing_period updates to yearly and FEAT-14.SPEC-010 sends the confirmation.

**FEAT-14.SPEC-008-AC-05:** Given Maya confirms a downgrade, when the current period's end date is reached, then tier is set to free, billing_period is set to none, billing_state is set to Active, and FEAT-23 becomes the household's planning route.

**FEAT-14.SPEC-008-AC-06:** Given Maya confirms a cancellation, when this automation fires, then billing_state is set to Cancelled immediately while tier and billing_period remain unchanged, paid features stay active, and no billing_history entry is created.

**FEAT-14.SPEC-008-AC-07:** Given FEAT-14.SPEC-007 invokes this automation after a 7-day unresolved grace period, when it fires, then tier is set to free, billing_period is set to none, and billing_state is set to Reverted to free.

**FEAT-14.SPEC-008-AC-08:** Given a household reverts to free by any path, when the reversion completes, then every past plan, rating, recipe, pantry item, and the shared list remain fully available and unaltered.

**FEAT-14.SPEC-008-AC-09:** Given Maya has a pending period switch and later confirms a downgrade before the switch's renewal date, when the downgrade's period-end date is reached, then tier moves to free and the pending period switch never applies.

**FEAT-14.SPEC-008-AC-10:** Given an upgrade payment succeeds while a stale downgrade confirmation from an earlier session also attempts to apply, when both are processed, then the one that reads current billing_state first proceeds and the second is refused and shown the current state.

**FEAT-14.SPEC-008-AC-11:** Given the Subscription write itself fails due to an internal processing error, when the failure occurs, then no partial write is committed and the Subscription record retains its prior, consistent state.

**FEAT-14.SPEC-008-AC-12:** Given a signal to FEAT-05 following a completed downgrade is not acknowledged, when FEAT-05 next checks tier gating on its own, then it reads the current, already-committed free tier directly from the Subscription record.

**FEAT-14.SPEC-008-AC-13:** Given a household with billing_state Cancelled reaches its current_period_end_date, when this automation applies the reversion, then tier is set to free, billing_period is set to none, billing_state is set to Reverted to free, billing_history records the change, and FEAT-14.SPEC-010 sends the reversion confirmation.

**FEAT-14.SPEC-008-AC-14:** Given Maya's upgrade payment succeeds, when this automation fires, then current_period_end_date is set to one billing_period ahead of the upgrade date.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 | 5 |
| Outcome Paths | 9 (upgrade applied, period switch scheduled, period switch applied, downgrade scheduled, downgrade applied, cancellation recorded, cancellation applied, grace-expiry applied, automation failure) | 9 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Integration Spec: Payment Processing Integration

## Overview

**Name:** Payment Processing Integration
**ID:** FEAT-14.SPEC-009
**Type:** Integration
**Purpose:** Product boundary to the payment-processing capability: submits payment methods, initiates and retries charges, and receives renewal outcome events.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Submitting a new payment method for an upgrade or a payment-details update
- Initiating the upgrade charge and subsequent recurring renewal charges
- Retrying a charge during the grace period
- Receiving renewal outcome events (success, failure) from the capability
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure of what payment and billing data is shared with the capability

**Non-Goals:**
- Choosing the payment vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Deciding grace-period timing or the no-partial-refund policy -- owned by FEAT-14.SPEC-005 (Billing State & Refund Rules); this spec only carries out the charges and retries those rules require
- Writing the resulting tier, billing_period, or billing_state changes to the Subscription record -- owned by FEAT-14.SPEC-008 (Apply Subscription Change), which this spec's outcome events feed into
- Processing payments for anything other than the household subscription -- product-features.md defines no other payable object in this product

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-33 -- "Payment-processing capability -- Required to run the paid household subscription (monthly or yearly)" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-14 -- the only feature on this row)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya enters payment details and subscribes to the paid tier | Upgrade to paid | FEAT-14.SPEC-002 (Upgrade to Paid) |
| Maya updates her payment method and sees billing history | Manage billing | FEAT-14.SPEC-003 (Billing & Payment Management) |
| A failed renewal charge is retried when Maya updates her payment details during the grace period | Manage billing (grace-period recovery) | FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) |
| The household's Subscription reflects the true outcome of every charge and renewal attempt | Upgrade to paid, Manage billing | FEAT-14.SPEC-008 (Apply Subscription Change) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Payment method details entered by Maya | Not stored as a Subscription field -- entered directly on FEAT-14.SPEC-002 or FEAT-14.SPEC-003 and passed through to the capability | Maya submits or updates payment details | The capability needs a payment method to charge |
| Charge amount and currency | Derived from the chosen billing_period (platform parameter: `subscription-price-monthly` or platform parameter: `subscription-price-yearly`) and Household -- currency | An upgrade, renewal, or retry charge is initiated | The capability must know what to charge and in what currency |
| Household reference | Household -- an internal reference only, not household content | Every charge or retry | Ties the payment outcome back to the correct household's Subscription |

No Dietary Rule, Weekly Plan, Grocery List, Recipe, or Pantry Item data -- nor any other Member Profile's data -- ever leaves the product through this integration. Only the organiser's own payment method and the household's billing reference are shared.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Charge outcome (succeeded / failed) | The capability reports the result of an upgrade, renewal, or retry charge | Subscription -- billing_state (via FEAT-14.SPEC-008 for success; via FEAT-14.SPEC-007 for a renewal failure) |
| Failure reason (plain-language category) | A charge fails | Subscription -- billing_history (recorded against the failed attempt) |
| Charged amount and date | A charge succeeds | Subscription -- billing_history |
| Payment method summary (masked) | Maya submits or updates payment details | Displayed on FEAT-14.SPEC-003; not stored as Subscription content beyond the masked summary needed to show what is on file |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Upgrade charge succeeded | Maya's upgrade payment is accepted | None directly -- feeds FEAT-14.SPEC-008 for the Subscription write | FEAT-14.SPEC-002 shows success and navigates to FEAT-14.SPEC-001 | FEAT-14.SPEC-002, FEAT-14.SPEC-008 |
| Upgrade charge failed | Maya's upgrade payment is declined or errors | None -- no Subscription change | FEAT-14.SPEC-002 preserves the chosen plan option and offers a retry, per this feature's Error state | FEAT-14.SPEC-002 |
| Renewal charge succeeded | A recurring renewal charge is accepted | billing_history entry appended | No direct user feedback beyond the ordinary billing history entry -- a successful renewal is the expected, silent case | FEAT-14.SPEC-003 |
| Renewal charge failed | A recurring renewal charge is declined or errors | Feeds FEAT-14.SPEC-007, which sets billing_state to Payment failed | Grace-period notice sent (FEAT-14.SPEC-011) | FEAT-14.SPEC-007, FEAT-14.SPEC-011 |
| Retry charge succeeded (during grace) | Maya's grace-period retry is accepted | Feeds FEAT-14.SPEC-007, which sets billing_state back to Active | Billing confirmation sent (FEAT-14.SPEC-010) | FEAT-14.SPEC-007, FEAT-14.SPEC-010 |
| Retry charge failed (during grace) | Maya's grace-period retry is declined or errors | None -- billing_state remains Payment failed | FEAT-14.SPEC-003 shows the retry failure inline; grace-period timer is unaffected | FEAT-14.SPEC-003, FEAT-14.SPEC-007 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-14.SPEC-002 (Upgrade to Paid) | "Subscribe" shows a loading state; after 10 seconds a note appears: "Still confirming -- this is taking longer than usual." | "Subscribe" is disabled with "Payment collection is temporarily unavailable. Try again in a few minutes." No charge is attempted and no Subscription change occurs. | The chosen plan option is preserved with the rejection reason in plain language: "This payment was declined: {reason}. Check your details and try again." |
| FEAT-14.SPEC-003 (Billing & Payment Management) | "Update payment details" or "Switch to {period}" shows a brief loading state; no separate slow-specific message, since these actions are not time-critical | The relevant action is disabled with "Payment collection is temporarily unavailable. Your current billing details are unchanged." The rest of the screen (viewing tier, billing history, downgrade/cancel navigation) remains usable. | Message with the rejection reason in plain language: "This update was declined: {reason}. Your previous payment details remain in effect." |
| FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling, retry path) | The retry attempt shows a brief in-progress state on FEAT-14.SPEC-003; no user action is blocked while waiting | The retry is queued and reattempted once the capability recovers, without narrowing the 7-day grace window itself -- the grace period's own timing (FEAT-14.SPEC-005) is independent of capability availability | The retry fails with the rejection reason shown on FEAT-14.SPEC-003; billing_state remains Payment failed and Maya may try again |

## Consent and Disclosure

- **First payment-details disclosure** -- The first time Maya enters payment details (on FEAT-14.SPEC-002), a notice appears before the payment form: "To subscribe, your payment details and household billing reference are shared with an external payment-processing service." Options: "Continue" and "Cancel". Shown once; afterwards a "How payment data is shared" link on FEAT-14.SPEC-003 reopens the same notice.
- **Renewal and retry disclosure** -- Recurring renewal charges and grace-period retries reuse the payment method already on file under the same original disclosure; no repeated prompt interrupts each renewal, since the organiser already consented to recurring billing at upgrade.
- **What is never shared** -- Dietary Rule data (including any child's allergy information), Weekly Plan and Grocery List content, Recipe data, Pantry Item data, and every other household member's data never leave the product through this integration; only Maya's own payment method and the household's billing reference are shared.

## Edge Cases

- **A charge outcome event arrives for a household that was deleted between the charge attempt and the event's delivery** -- The event is discarded silently; no Subscription write occurs for a deleted household, and no user feedback fires.
- **The same renewal-failure event is delivered twice** -- The second delivery changes nothing: FEAT-14.SPEC-007 already recorded billing_state as Payment failed with the same failure date, and no duplicate grace-period notice is sent.
- **Events arrive out of order (a renewal-success event for a later period arrives before an earlier renewal-failure event for the same Subscription)** -- The Subscription reflects the most recent event by its own event time, not arrival time; a late-arriving failure event for an already-superseded period is discarded, since a later successful renewal has already resolved billing_state to Active.
- **Capability goes down mid-charge for an upgrade** -- If the charge was not confirmed initiated, FEAT-14.SPEC-002 shows the capability-down message and no Subscription change occurs -- no half-completed upgrade state exists.
- **Maya updates payment details while a renewal charge for the same Subscription is already in flight** -- The in-flight renewal charge completes against the payment method that was on file when it was initiated; Maya's update takes effect for the next charge attempt only, avoiding a mid-charge method swap.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-002 (Upgrade to Paid) | Triggered by (inbound) | "Subscribe" initiates the upgrade charge |
| FEAT-14.SPEC-002 (Upgrade to Paid) | Affects (outbound) | Charge outcome and degradation states surface here |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Triggered by (inbound) | Payment-details updates and billing-history reads initiate here |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Affects (outbound) | Charge outcome, billing history, and degradation states surface here |
| FEAT-14.SPEC-005 (Billing State & Refund Rules) | References (inbound) | Payment-validity format rules this spec enforces |
| FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Triggers (outbound) | Renewal-failure and retry-outcome events fire this automation |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | A successful upgrade or renewal charge feeds this automation's write |

## Analytics and Success Signals

- **payment_method_submitted** (context: upgrade / update) -- supports success-metrics.md: "Paid Conversion Rate"
- **charge_outcome_received** (context: upgrade / renewal / retry; outcome: succeeded / failed) -- supports success-metrics.md: "Paid Conversion Rate"
- **payment_degradation_shown** (condition: slow / down / rejected; screen: spec ID) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for capability trouble is observable.

## Acceptance Criteria

**FEAT-14.SPEC-009-AC-01:** Given Maya enters valid payment details on FEAT-14.SPEC-002 and selects Monthly, when she taps Subscribe, then this integration submits a charge for the platform parameter: `subscription-price-monthly` amount in her household's currency.

**FEAT-14.SPEC-009-AC-02:** Given the upgrade charge succeeds, when the outcome is received, then FEAT-14.SPEC-008 is fed the success and writes tier=paid.

**FEAT-14.SPEC-009-AC-03:** Given the upgrade charge is declined, when the outcome is received, then FEAT-14.SPEC-002 preserves Maya's chosen plan option and offers a retry.

**FEAT-14.SPEC-009-AC-04:** Given a recurring renewal charge fails, when the outcome is received, then FEAT-14.SPEC-007 is fed the failure and opens the grace period.

**FEAT-14.SPEC-009-AC-05:** Given Maya updates payment details during a grace period and the retry succeeds, when the outcome is received, then FEAT-14.SPEC-007 clears the grace state.

**FEAT-14.SPEC-009-AC-06:** Given Maya taps Subscribe while the capability is unavailable, when the request cannot be sent, then the button is disabled with "Payment collection is temporarily unavailable. Try again in a few minutes." and no charge is attempted.

**FEAT-14.SPEC-009-AC-07:** Given the capability rejects a payment-details update on FEAT-14.SPEC-003, when this occurs, then the rejection reason is shown in plain language and her previous payment details remain in effect.

**FEAT-14.SPEC-009-AC-08:** Given Maya has never entered payment details before, when she reaches the payment form on FEAT-14.SPEC-002, then the data-sharing notice appears with "Continue" and "Cancel", and no data leaves the product until she chooses "Continue".

**FEAT-14.SPEC-009-AC-09:** Given a recurring renewal charge succeeds, when the outcome is received, then no repeated consent prompt interrupts Maya, since renewal reuses the original disclosure.

**FEAT-14.SPEC-009-AC-10:** Given a charge-outcome event arrives for a household deleted since the charge attempt, when this integration processes it, then no Subscription write occurs and no user feedback fires.

**FEAT-14.SPEC-009-AC-11:** Given the same renewal-failure event is delivered twice, when the second delivery arrives, then billing_state is unchanged and no duplicate grace-period notice fires.

**FEAT-14.SPEC-009-AC-12:** Given a late-arriving renewal-failure event for a period already superseded by a later successful renewal, when it is processed, then it is discarded and billing_state remains Active.

**FEAT-14.SPEC-009-AC-13:** Given the capability goes down mid-charge for an upgrade that was not confirmed initiated, when Maya checks her tier, then it remains free -- no half-completed upgrade state exists.

**FEAT-14.SPEC-009-AC-14:** Given Maya updates her payment details while an in-flight renewal charge is already processing against the prior method, when the in-flight charge completes, then it completes against the payment method on file when it was initiated, and Maya's update applies only to the next charge attempt.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 6 | 6 |
| Degradation Paths | 9 (3 screens x 3 conditions) | 9 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |



# Notification Spec: Billing Confirmation Notification

## Overview

**Name:** Billing Confirmation Notification
**ID:** FEAT-14.SPEC-010
**Type:** Notification
**Purpose:** Sends confirmation of an upgrade, downgrade, cancellation, period switch, or grace-period resolution to the organiser.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- The confirmation sent when an upgrade completes
- The confirmation sent when a billing-period switch takes effect
- The confirmation sent when a downgrade or cancellation reversion completes
- The confirmation sent when a grace-period retry succeeds or a grace period expires unresolved

**Non-Goals:**
- The initial grace-period notice itself -- owned by FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice); this spec covers only the confirmation once the grace period is resolved (successfully or by lapsing)
- Deciding which changes are applied and when -- owned by FEAT-14.SPEC-008 (Apply Subscription Change); this spec begins where that automation's trigger fires
- The transactional email capability's own send/delivery mechanics -- owned by FEAT-14.SPEC-012 (Transactional Email Delivery (Billing)), which this notification's email channel is delivered through

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a billing change is confirmed | Maya reviews her plan and billing inside the product regularly; the confirmation belongs where the change is visible |
| Email | Always, in addition to in-app | Billing changes are consequential enough to warrant a durable record outside the session in which they occurred, and Maya's Sunday-evening usage pattern means she is often away from the product when a renewal-driven change (a period switch or a grace-period resolution) actually takes effect |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Upgrade applied | FEAT-14.SPEC-008 (Apply Subscription Change) | Fires when an upgrade is written to the Subscription record | Household reference, billing_period, currency |
| Period switch applied | FEAT-14.SPEC-008 (Apply Subscription Change) | Fires when a scheduled period switch takes effect at renewal | Household reference, new billing_period |
| Downgrade or cancellation applied | FEAT-14.SPEC-008 (Apply Subscription Change) | Fires when a scheduled downgrade or cancellation reversion completes at period end | Household reference, reversion type (downgrade / cancellation) |
| Grace-period retry succeeded | FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Fires when a grace-period retry charge succeeds and billing_state returns to Active | Household reference |
| Grace-period expired unresolved | FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) via FEAT-14.SPEC-008 | Fires when the 7-day grace period lapses and the household reverts to free | Household reference |

## Audience and Preferences

**Recipients:** Maya -- the organiser, per the Billing column of the Access Matrix in user-persona.md (Full). No other household role receives this notification: Sam and both Jordan rows have Billing: None, and Riley's Billing access is View of plan tier only and never includes notifications about it.

**Preference Controls:**

N/A -- this notification carries no user-configurable preference. Member Profile's notification_preferences field (per the Feature Dependency Map) covers only the plan-ready and nightly-nudge toggles owned by FEAT-07 and FEAT-13; the product defines no corresponding on/off control for billing confirmations. Both channels (in-app and email) always fire for every applied change, since a billing confirmation is a required record of a consequential account change, not a discretionary alert Maya can silence.

**Quiet Hours:** N/A -- billing confirmations are not held for quiet hours. A billing change is a direct consequence of Maya's own action (upgrade, period switch, downgrade, cancellation) or a resolution she is actively waiting on (grace-period outcome), so delaying the confirmation would leave her without timely proof that her action took effect.

## Content Definition

**In-app:**
- **Title (upgrade):** You're on the paid plan
- **Body (upgrade):** Your {billing_period} subscription is active. Your first AI-generated plan is on its way.
- **CTA:** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

- **Title (period switch):** Your billing period changed
- **Body (period switch):** You're now on {billing_period} billing, effective this renewal.
- **CTA:** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Title (downgrade/cancellation applied):** You're on the free plan
- **Body (downgrade/cancellation applied):** Your household moved to the free plan. Every past plan, rating, recipe, pantry item, and your shared list are still fully available.
- **CTA:** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

- **Title (grace-period retry succeeded):** Your payment went through
- **Body (grace-period retry succeeded):** Your subscription is active again -- no interruption to your paid features.
- **CTA:** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Title (grace-period expired):** You're on the free plan
- **Body (grace-period expired):** We couldn't complete your renewal payment, so your household moved to the free plan. Every past plan, rating, recipe, pantry item, and your shared list are still fully available.
- **CTA:** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

**Email:**
- **Subject (upgrade):** You're subscribed to Plateful {billing_period}
- **Body (upgrade):**
  Hi {organiser_first_name},

  Your {billing_period} Plateful subscription is now active. Your first AI-generated weekly plan is on its way.

  Manage your billing anytime from your account.
- **CTA (button):** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Subject (period switch):** Your Plateful billing period changed
- **Body (period switch):**
  Hi {organiser_first_name},

  You're now on {billing_period} billing for Plateful, effective this renewal.
- **CTA (button):** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Subject (downgrade/cancellation applied):** Your Plateful household is now on the free plan
- **Body (downgrade/cancellation applied):**
  Hi {organiser_first_name},

  Your household has moved to the free Plateful plan. Every past plan, rating, recipe, pantry item, and your shared list are still fully available -- nothing has been removed.

  You can plan manually anytime, or upgrade again whenever you're ready.
- **CTA (button):** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

- **Subject (grace-period retry succeeded):** Your Plateful payment went through
- **Body (grace-period retry succeeded):**
  Hi {organiser_first_name},

  Your payment was successful and your subscription is active again. There's no interruption to your paid features.
- **CTA (button):** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Subject (grace-period expired):** Your Plateful household is now on the free plan
- **Body (grace-period expired):**
  Hi {organiser_first_name},

  We weren't able to complete your renewal payment, so your household has moved to the free plan. Every past plan, rating, recipe, pantry item, and your shared list are still fully available -- nothing has been removed.

  You can resubscribe anytime.
- **CTA (button):** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {billing_period} | Subscription -- billing_period | Yearly | Never empty -- every trigger for this notification fires only once billing_period is a concrete value (monthly or yearly) or, for downgrade/cancellation/grace-expiry, is not referenced in that variant's content |
| {organiser_first_name} | Member Profile -- display_name (of the organiser) | Maya | Greeting renders as "Hi there,"

## Delivery Rules

**Batching:** No batching -- each billing change produces exactly one notification instance, since a household's Subscription changes at most once per confirmed action and these confirmations are consequential enough to warrant individual delivery rather than being combined.
**Deduplication:** At most one confirmation per applied change. FEAT-14.SPEC-008 and FEAT-14.SPEC-007 each fire this notification exactly once per outcome they apply; a re-run of either automation for an already-applied change (per their own idempotency rules) never re-triggers this notification.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-14.SPEC-012. After the final failure, the in-app confirmation stands as the delivery of record and no additional error is shown to Maya -- a billing confirmation must never generate an alarming failure message of its own.
**Expiry:** The in-app confirmation never expires undelivered -- it is delivered the next time Maya opens the product, since it reflects a durable state change (the current tier and billing_state) rather than a time-sensitive alert. The email variant follows FEAT-14.SPEC-012's own retry-and-give-up behavior; if it is never delivered, the in-app confirmation and the visible tier on FEAT-14.SPEC-001 remain the surviving record.

## Edge Cases

- **Household deleted between the change applying and delivery** -- The notification is cancelled silently on every channel; a confirmation about a household that no longer exists is never delivered.
- **Maya has no registered email on her Member Profile** -- The email channel is skipped for that delivery; the in-app confirmation still delivers as the primary record.
- **A downgrade confirmation and a period-switch confirmation would otherwise both fire for the same Subscription in the same processing window (a downgrade supersedes a pending period switch, per FEAT-14.SPEC-008)** -- Only the downgrade confirmation is sent; the period-switch confirmation is never fired for a switch that FEAT-14.SPEC-008 determined never applied.
- **Grace-period retry succeeds at effectively the same moment the grace-expiry reversion would otherwise have fired** -- Per FEAT-14.SPEC-007's own resolution, only one outcome is applied; only that outcome's confirmation content (retry-succeeded or expired) is sent, never both.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggered by (inbound) | Every applied tier or billing_period change fires this notification |
| FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Triggered by (inbound) | A grace-period retry success or expiry fires this notification |
| FEAT-14.SPEC-012 (Transactional Email Delivery (Billing)) | References (outbound) | The email channel is delivered through this integration |
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (outbound) | Upgrade, downgrade, and grace-expiry confirmations deep-link here |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Navigation (outbound) | Period-switch and grace-retry-success confirmations deep-link here |

## Analytics and Success Signals

- **billing_confirmation_delivered** (channel: in_app / email; variant: upgrade / period_switch / downgrade / cancellation / grace_retry / grace_expiry) -- supports success-metrics.md: "Paying Household Retention"
- **billing_confirmation_cta_tapped** (channel; destination: plan_tier_overview / billing_management) -- supports success-metrics.md: "Paying Household Retention"
- **billing_confirmation_email_skipped** (reason: no_email_on_file) -- N/A -- no Stage 2 metric tracks skipped email deliveries; retained so silent delivery gaps remain observable

## Acceptance Criteria

**FEAT-14.SPEC-010-AC-01:** Given Maya's upgrade completes, when FEAT-14.SPEC-008 fires, then she receives an in-app notification titled "You're on the paid plan" and an email with the subject "You're subscribed to Plateful {billing_period}".

**FEAT-14.SPEC-010-AC-02:** Given Maya taps the upgrade confirmation's CTA, when she taps "View plan", then she lands on FEAT-14.SPEC-001.

**FEAT-14.SPEC-010-AC-03:** Given a scheduled period switch takes effect at renewal, when FEAT-14.SPEC-008 applies it, then Maya receives the period-switch confirmation naming the new billing period.

**FEAT-14.SPEC-010-AC-04:** Given a downgrade reversion completes at period end, when FEAT-14.SPEC-008 applies it, then Maya receives "You're on the free plan" stating that all past data remains fully available.

**FEAT-14.SPEC-010-AC-05:** Given a cancellation reversion completes at period end, when FEAT-14.SPEC-008 applies it, then Maya receives the same free-plan confirmation content as a downgrade.

**FEAT-14.SPEC-010-AC-06:** Given a grace-period retry succeeds, when FEAT-14.SPEC-007 clears the grace state, then Maya receives "Your payment went through" confirming no interruption to paid features.

**FEAT-14.SPEC-010-AC-07:** Given a grace period expires unresolved, when the household reverts to free, then Maya receives the grace-expiry confirmation stating the reversion and that all past data remains available.

**FEAT-14.SPEC-010-AC-08:** Given the email delivery for a confirmation fails 3 times over 6 hours, when the final retry fails, then no error is shown to Maya and the in-app confirmation stands as the record.

**FEAT-14.SPEC-010-AC-09:** Given a household is deleted between a change applying and this notification's delivery, when delivery would otherwise occur, then it is cancelled silently on every channel.

**FEAT-14.SPEC-010-AC-10:** Given a pending period switch is superseded by a confirmed downgrade in the same processing window, when FEAT-14.SPEC-008 resolves both, then only the downgrade confirmation is sent.

**FEAT-14.SPEC-010-AC-11:** Given a grace-period retry succeeds at effectively the same moment expiry would otherwise fire, when FEAT-14.SPEC-007 resolves the outcome, then only that single outcome's confirmation is sent, never both.

**FEAT-14.SPEC-010-AC-12:** Given Maya has no registered email on her Member Profile, when a billing change is confirmed, then only the in-app notification is delivered.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 5 | 5 |
| Preference States | 1 (no configurable preference -- always-on both channels) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |



# Notification Spec: Payment Failure Grace-Period Notice

## Overview

**Name:** Payment Failure Grace-Period Notice
**ID:** FEAT-14.SPEC-011
**Type:** Notification
**Purpose:** Sends the organiser a clear grace-period notice and a path to update payment details after a failed renewal, distinct in urgency from a routine billing confirmation.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- The notice sent the moment a renewal payment failure opens the 7-day grace period
- Its exact content on every channel it uses, including the path to update payment details

**Non-Goals:**
- The confirmation sent once the grace period is resolved (retry success or expiry) -- owned by FEAT-14.SPEC-010 (Billing Confirmation Notification); this spec covers only the initial notice
- Deciding the grace period's length or timing -- owned by FEAT-14.SPEC-005 (Billing State & Refund Rules); this spec only communicates the outcome of that rule
- Retrying the charge itself -- owned by FEAT-14.SPEC-009 (Payment Processing Integration), triggered from FEAT-14.SPEC-003 where this notice's CTA leads
- The transactional email capability's own send/delivery mechanics -- owned by FEAT-14.SPEC-012 (Transactional Email Delivery (Billing)), which this notification's email channel is delivered through

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a grace period opens | Maya needs the persistent grace/billing-state banner reinforced by an explicit notice she cannot miss on her next visit |
| Email | Always, in addition to in-app | A renewal failure can occur at any time, including while Maya is away from the product for days; email is the channel most likely to reach her before the 7-day grace window narrows, consistent with the urgency this notice carries |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Grace period opened | FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Fires immediately when a renewal payment failure sets billing_state to Payment failed | Household reference, payment_failure_date (grace-period end date = payment_failure_date + 7 days) |

## Audience and Preferences

**Recipients:** Maya -- the organiser, per the Billing column of the Access Matrix in user-persona.md (Full). No other household role receives this notice: Sam and both Jordan rows have Billing: None, and Riley's Billing access is View of plan tier only and never includes payment-related notices.

**Preference Controls:**

N/A -- this notice carries no user-configurable preference, for the same reason as FEAT-14.SPEC-010: it is a required, time-sensitive account notice about the household's own payment status, not a discretionary alert Maya can silence. Both channels always fire.

**Quiet Hours:** N/A -- a renewal payment failure is not held for quiet hours. Delaying this notice would shorten Maya's effective window to act within the 7-day grace period without shortening the grace period itself, working directly against the notice's purpose.

## Content Definition

**In-app:**
- **Title:** Payment failed -- action needed
- **Body:** We couldn't process your renewal payment. Update your card by {grace_period_end_date} to keep your paid features.
- **CTA:** Update payment details -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

**Email:**
- **Subject:** Action needed: your Plateful payment didn't go through
- **Body:**
  Hi {organiser_first_name},

  We weren't able to process your renewal payment for Plateful. Your paid features are still active, but you'll need to update your payment details by {grace_period_end_date} to avoid moving to the free plan.

  If nothing changes by then, your household moves to the free plan -- no data is lost. Every past plan, rating, recipe, pantry item, and your shared list will still be fully available.
- **CTA (button):** Update payment details -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {grace_period_end_date} | Derived -- Subscription.payment_failure_date + 7 days, computed by FEAT-14.SPEC-007 | March 14 | Never empty -- this notice is fired only once FEAT-14.SPEC-007 has recorded payment_failure_date and computed the grace-period end date |
| {organiser_first_name} | Member Profile -- display_name (of the organiser) | Maya | Greeting renders as "Hi there," |

## Delivery Rules

**Batching:** No batching -- exactly one grace period can be open per household at a time (a household cannot enter a second grace period while already in one), so no scenario produces multiple pending instances to combine.
**Deduplication:** At most one notice per grace-period opening. A duplicate renewal-failure event for the same failure (per FEAT-14.SPEC-009's Edge Cases) never re-triggers this notice, since FEAT-14.SPEC-007 only opens the grace period once per failure.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-14.SPEC-012. After the final failure, the in-app notice and the persistent billing-state banner (FEAT-14.SPEC-001, FEAT-14.SPEC-003) stand as the surviving record -- Maya still sees the grace state every time she opens the product even if the email never arrives.
**Expiry:** This notice does not expire in the usual sense -- it remains relevant for the entire 7-day grace period. If it has not been delivered by the time the grace period resolves (retry succeeds or expiry occurs), the pending send is superseded by FEAT-14.SPEC-010's resolution confirmation rather than being sent late as a now-irrelevant "you're in a grace period" notice.

## Edge Cases

- **The grace period resolves (retry succeeds or expiry occurs) before this notice's email has been delivered** -- The pending email send is cancelled; delivering a "your payment failed, act by {date}" notice after the grace period has already resolved would be actively confusing, so FEAT-14.SPEC-010's resolution confirmation takes its place.
- **Household is deleted while a grace period is open and this notice is still pending delivery** -- The notice is cancelled silently on every channel.
- **Maya has no registered email on her Member Profile** -- The email channel is skipped for that delivery; the in-app notice and billing-state banner still deliver as the primary record.
- **The same renewal-failure event is delivered twice (per FEAT-14.SPEC-009's Edge Cases)** -- Only one notice is ever sent, since FEAT-14.SPEC-007 sets billing_state to Payment failed only once for the same failure.
- **Maya updates her payment details and the retry fails, still within the grace period** -- This notice is not re-sent; the grace period and its end date are unchanged, and the retry failure is shown inline on FEAT-14.SPEC-003 rather than through a repeated notice.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Triggered by (inbound) | Grace-period opening fires this notice |
| FEAT-14.SPEC-012 (Transactional Email Delivery (Billing)) | References (outbound) | The email channel is delivered through this integration |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Navigation (outbound) | The CTA deep-links here to update payment details |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | References (outbound) | The eventual resolution confirmation supersedes this notice once the grace period resolves |

## Analytics and Success Signals

- **grace_period_notice_delivered** (channel: in_app / email) -- supports success-metrics.md: "Paying Household Retention"
- **grace_period_notice_cta_tapped** (channel) -- supports success-metrics.md: "Paying Household Retention"
- **grace_period_notice_email_skipped** (reason: no_email_on_file) -- N/A -- no Stage 2 metric tracks skipped email deliveries; retained so silent delivery gaps remain observable

## Acceptance Criteria

**FEAT-14.SPEC-011-AC-01:** Given a renewal payment failure opens a grace period for Maya's household, when FEAT-14.SPEC-007 fires, then she receives an in-app notice titled "Payment failed -- action needed" and an email with the subject "Action needed: your Plateful payment didn't go through".

**FEAT-14.SPEC-011-AC-02:** Given Maya receives this notice, when she taps "Update payment details", then she lands on FEAT-14.SPEC-003 (Billing & Payment Management).

**FEAT-14.SPEC-011-AC-03:** Given the notice is delivered, when Maya reads it, then it states the exact date by which she must act, computed as 7 days from the failure date.

**FEAT-14.SPEC-011-AC-04:** Given Maya's grace period resolves via a successful retry before this notice's email has sent, when the retry succeeds, then the pending email send is cancelled and FEAT-14.SPEC-010's confirmation is sent instead.

**FEAT-14.SPEC-011-AC-05:** Given Maya's grace period expires unresolved before this notice's email has sent, when the expiry occurs, then the pending email send is cancelled and FEAT-14.SPEC-010's expiry confirmation is sent instead.

**FEAT-14.SPEC-011-AC-06:** Given the household is deleted while this notice is pending, when the deletion completes, then the notice is cancelled silently on every channel.

**FEAT-14.SPEC-011-AC-07:** Given Maya has no registered email on her Member Profile, when the grace period opens, then only the in-app notice is delivered.

**FEAT-14.SPEC-011-AC-08:** Given the same renewal-failure event is delivered twice, when the second delivery arrives, then only one notice was ever sent.

**FEAT-14.SPEC-011-AC-09:** Given Maya updates her payment details and the retry fails within the same grace period, when the retry outcome is processed, then this notice is not re-sent and the grace-period end date is unchanged.

**FEAT-14.SPEC-011-AC-10:** Given the email delivery for this notice fails 3 times over 6 hours, when the final retry fails, then the in-app notice and the persistent billing-state banner remain the surviving record with no additional error shown.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no configurable preference -- always-on both channels) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Integration Spec: Transactional Email Delivery (Billing)

## Overview

**Name:** Transactional Email Delivery (Billing)
**ID:** FEAT-14.SPEC-012
**Type:** Integration
**Purpose:** Product boundary to the transactional email capability used to deliver billing confirmations and the grace-period notice.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Sending the billing confirmation email (FEAT-14.SPEC-010's email variants)
- Sending the payment-failure grace-period notice email (FEAT-14.SPEC-011)
- User-facing behavior when the transactional email capability is slow, unavailable, or rejects a send
- Disclosure of what billing data is shared with the capability to deliver these emails

**Non-Goals:**
- Choosing the email-delivery vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Any other transactional email this product sends (account/recovery, safety reports, plan-ready fallback, export/deletion/support-acknowledgement emails) -- each is owned by the feature whose Communications require it (FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-18.SPEC-012 respectively, per the Feature Dependency Map's External Touchpoints table); this spec covers only the billing-related emails named in this feature's own Communications
- Deciding the exact wording of each email's subject and body -- owned by FEAT-14.SPEC-010 and FEAT-14.SPEC-011, whose Content Definition sections this spec delivers verbatim
- The in-app channel for either notification -- owned by FEAT-14.SPEC-010 and FEAT-14.SPEC-011 directly; this spec covers only the email channel

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Transactional email capability -- Required for ... billing and grace-period notices ..." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this spec, FEAT-14.SPEC-012, covers the billing-confirmation and grace-period-notice portion)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya receives an email confirming an upgrade, period switch, downgrade, cancellation, or grace-period resolution | Upgrade to paid, Manage billing | FEAT-14.SPEC-010 (Billing Confirmation Notification) |
| Maya receives an email notice when a renewal payment fails, with a path to update payment details | Manage billing | FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Organiser's email address | Member Profile -- sign_in (email component, organiser only) | Any FEAT-14.SPEC-010 or FEAT-14.SPEC-011 email is triggered | The capability needs a destination address to deliver the email |
| Organiser's first name | Member Profile -- display_name (organiser) | Same as above | Personalizes the greeting, per the exact content templates in FEAT-14.SPEC-010 and FEAT-14.SPEC-011 |
| Billing content (tier, billing_period, grace-period end date) | Subscription -- tier, billing_period; derived grace-period end date | Same as above | The email's exact content, per FEAT-14.SPEC-010 and FEAT-14.SPEC-011's Content Definition sections |

No payment method details, billing_history amounts beyond what the confirmation content itself states, Dietary Rule data, or any other household member's data ever leaves the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / bounced / failed) | The capability reports the send result | No entity field is updated by a successful delivery; a bounced or failed send updates an internal delivery-status flag on the pending email attempt (not a Household, Member Profile, or Subscription field), used only to decide whether to retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Billing confirmation email delivered | The capability confirms the email reached the recipient's inbox | None -- delivery confirmation is not surfaced as a user-visible change | None -- FEAT-14.SPEC-010's in-app notification already delivered regardless of email delivery status | FEAT-14.SPEC-010 |
| Grace-period notice email delivered | The capability confirms the email reached the recipient's inbox | None | None -- FEAT-14.SPEC-011's in-app notice already delivered regardless of email delivery status | FEAT-14.SPEC-011 |
| Send failed | The capability reports it could not deliver either email (bounced, rejected, or a hard failure) | The pending email attempt's internal delivery-status flag is set to failed | None immediately -- see Degradation Behavior; the triggering notification's in-app channel is unaffected | FEAT-14.SPEC-010, FEAT-14.SPEC-011 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | No user-visible effect -- the in-app confirmation appears regardless of email send speed | No user-visible effect -- the in-app confirmation is unaffected; the email is queued to send once the capability recovers, up to its own retry limit | No user-visible effect on Maya's session -- a rejected send does not block or alter the in-app confirmation, since email is a durable-record channel, not a required verification gate |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | No user-visible effect -- the in-app notice and billing-state banner appear regardless of email send speed | No user-visible effect on the in-app channel; the email is queued to send once the capability recovers, within the notice's own pending-then-superseded window (FEAT-14.SPEC-011's Delivery Rules) | No user-visible effect on Maya's session -- the in-app notice and billing-state banner remain the primary, always-delivered record of the grace period |

## Consent and Disclosure

- **Billing-email disclosure** -- Sending a billing confirmation or grace-period notice email to the organiser's own registered address is standard product behavior for a consequential account change; no separate consent prompt interrupts an upgrade, period switch, downgrade, cancellation, or grace-period event, since the organiser already provided that email address for exactly this kind of account communication (per FEAT-01.SPEC-017's account-level disclosure).
- **What is never shared** -- Payment method details, the full billing_history list, Dietary Rule data (including any child's allergy information), and every other household member's data never leave the product through this integration; only the organiser's own email, first name, and the specific billing content named in Data Exchanged are included.

## Edge Cases

- **A billing-email send event arrives for a household that was deleted between the trigger and the send** -- The event is discarded silently; no email is sent for a deleted household, and no user feedback fires, since the in-app channel for that household no longer exists either.
- **The same "send failed" event is delivered twice for one attempt** -- The second delivery changes nothing: the delivery-status flag is already failed, and no duplicate retry beyond the standard policy is triggered.
- **Confirmation and failure events arrive out of order (failure reported, then a late "delivered" event for the same attempt)** -- The most recent event by its own timestamp governs the delivery-status flag; a late "delivered" event arriving after a "failed" event corrects the flag back to delivered, since it reflects a true, if delayed, outcome.
- **Capability goes down mid-send for a grace-period notice** -- If the send was not confirmed initiated, it is treated as not yet sent and is queued for retry once the capability recovers, within FEAT-14.SPEC-011's own pending-then-superseded window; the in-app notice is unaffected either way.
- **A grace-period notice email is still queued when the grace period resolves** -- Per FEAT-14.SPEC-011's Edge Cases, the queued send is cancelled rather than delivered late, since the resolution confirmation (FEAT-14.SPEC-010) supersedes it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Triggered by (inbound) | Every confirmation variant's email channel is sent through this integration |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Affects (outbound) | Degradation behavior surfaces here (as no user-visible effect, by design) |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | Triggered by (inbound) | The grace-period notice's email channel is sent through this integration |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | Affects (outbound) | Degradation behavior surfaces here (as no user-visible effect, by design) |

## Analytics and Success Signals

- **billing_email_sent** (notification: confirmation / grace_notice; delivery outcome: delivered / failed) -- supports success-metrics.md: "Paying Household Retention" (a household unreachable by email during a grace period is at greater risk of an unresolved lapse this metric tracks)
- **billing_email_degradation_shown** (condition: slow / down / rejected; notification: confirmation / grace_notice) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for capability trouble is observable

## Acceptance Criteria

**FEAT-14.SPEC-012-AC-01:** Given Maya's upgrade completes, when FEAT-14.SPEC-010 fires, then this integration sends the upgrade-confirmation email to her registered address.

**FEAT-14.SPEC-012-AC-02:** Given a renewal payment failure opens a grace period, when FEAT-14.SPEC-011 fires, then this integration sends the grace-period notice email to Maya's registered address.

**FEAT-14.SPEC-012-AC-03:** Given the transactional email capability is slow, when a billing confirmation is triggered, then Maya's in-app confirmation is unaffected by the delay.

**FEAT-14.SPEC-012-AC-04:** Given the transactional email capability is down, when the grace-period notice is triggered, then Maya still sees the in-app notice and billing-state banner, and the email is queued to send once the capability recovers.

**FEAT-14.SPEC-012-AC-05:** Given the transactional email capability rejects a billing-confirmation send, when this occurs, then Maya's in-app confirmation and her session are unaffected.

**FEAT-14.SPEC-012-AC-06:** Given a billing email send event arrives for a household deleted since the trigger, when this integration processes it, then no email is sent and no user feedback fires.

**FEAT-14.SPEC-012-AC-07:** Given a "send failed" event for a billing email is delivered twice, when the second delivery arrives, then the delivery-status flag remains failed and no duplicate retry beyond the standard policy occurs.

**FEAT-14.SPEC-012-AC-08:** Given a "delivered" event arrives after an earlier "failed" event for the same billing-email attempt, when it is processed, then the delivery-status flag corrects to delivered.

**FEAT-14.SPEC-012-AC-09:** Given the capability goes down mid-send for a grace-period notice that was not confirmed initiated, when this occurs, then the send is queued for retry and the in-app notice is unaffected.

**FEAT-14.SPEC-012-AC-10:** Given a grace-period notice email is still queued when the grace period resolves, when the resolution occurs, then the queued send is cancelled rather than delivered late.

**FEAT-14.SPEC-012-AC-11:** Given Maya has never seen a data-sharing prompt interrupt a billing action, when she receives a billing email, then she recognizes it as standard account communication to the address she already provided, per FEAT-01.SPEC-017's account-level disclosure.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 6 (2 notifications x 3 conditions) | 6 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |
