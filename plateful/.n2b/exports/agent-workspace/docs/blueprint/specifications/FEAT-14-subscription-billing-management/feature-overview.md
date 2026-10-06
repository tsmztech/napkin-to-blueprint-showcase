---
document_type: feature-overview
feature_number: FEAT-14
feature_name: Subscription & Billing Management
feature_slug: subscription-billing-management
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 12
screen_count: 4
automation_count: 2
logic_rule_count: 2
integration_count: 2
notification_count: 2
---

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
