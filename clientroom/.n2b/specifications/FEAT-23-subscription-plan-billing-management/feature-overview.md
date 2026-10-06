---
document_type: feature-overview
feature_number: FEAT-23
feature_name: Subscription Plan & Billing Management
feature_slug: subscription-plan-billing-management
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 8
screen_count: 1
automation_count: 4
logic_rule_count: 1
integration_count: 1
notification_count: 1
---

# Feature Breakdown Brief: Subscription Plan & Billing Management

## Summary

**Feature:** Subscription Plan & Billing Management
**ID:** FEAT-23
**Description:** The freelancer sees her current Clientroom plan (free for one or two clients, a flat monthly or yearly price above that) and upgrades when she adds more clients.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** BRIEF.md, Business Context: "sold as a subscription per freelancer, priced by number of active clients: free for one or two clients, then one flat monthly or yearly price." Ranked Important rather than Core because it governs the freelancer's own relationship with the product, not the client-facing value loop; phased MVP because the founder's three-month first-paying-freelancer goal (BRIEF.md, Constraints) requires billing to exist at launch. [CHALLENGED: all 5 profiled competitors offer only a time-limited free trial (7-30 days) and no ongoing free tier (vendor pricing pages, 5 sources, HIGH confidence) -- original retained per SYN-04 protection (user-stated pricing model in BRIEF.md, Business Context); the free tier is also the entry point of the portal-driven growth loop] [RESEARCH-INFORMED: flat pricing without per-seat or per-contact charges draws repeated praise (SuiteDash, MEDIUM), so client contacts are unlimited on every plan] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- View current plan -- see tier and usage against its client limit
- Upgrade -- subscribe to the paid tier when exceeding the free-tier client count
- Downgrade offer -- reduce plan when client count drops back below the threshold
- Cancel -- stop the paid plan at any time, effective at the end of the paid period [AUDIT-ADDED: 3 -- entity coverage: Subscription Plan had no cancel path]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-23.SPEC-001 | Plan & Billing Screen | Screen | Nadia (Freelancer) | Nadia views her current tier, usage against its client limit, and billing cycle, and initiates upgrade, responds to a downgrade offer, or cancels, from one surface |
| FEAT-23.SPEC-002 | Free Plan Auto-Provisioning | Automation | Nadia (Freelancer) | Creates the Subscription Plan record on the free tier automatically the instant a new Freelancer Account is created, with no explicit "no plan" state |
| FEAT-23.SPEC-003 | Subscription Billing Processing | Integration | Nadia (Freelancer) | Submits upgrade, downgrade, and cancellation changes to the subscription-billing capability and receives back charge outcomes, renewal and period-end events, and failure reasons |
| FEAT-23.SPEC-004 | Plan State Sync | Automation | Nadia (Freelancer) | Applies the subscription-billing capability's reported outcomes and period-end/renewal events to the Subscription Plan record's tier and status |
| FEAT-23.SPEC-005 | Downgrade Eligibility Detection | Automation | Nadia (Freelancer) | Evaluates the active-client count against the free-tier threshold whenever it changes, and raises the downgrade offer on a Paid plan that has dropped back below the threshold |
| FEAT-23.SPEC-006 | Cancel Subscription | Automation | Nadia (Freelancer) | Records Nadia's cancellation with the plan remaining Paid and fully usable through the end of the current paid period |
| FEAT-23.SPEC-007 | Plan Limit & Access Authorization Rules | Logic/Rule | Nadia (Freelancer), Dana (Support Operator) | Governs the free-tier client cap gating capacity, who may view versus change the plan, and the grace/retry handling before a failed charge lapses the plan |
| FEAT-23.SPEC-008 | Plan & Billing Notifications | Notification | Nadia (Freelancer) | Sends Nadia an email confirmation on every plan change (upgrade, downgrade, cancellation, lapse) and an alert when a subscription charge fails |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| View current plan -- see tier and usage against its client limit | FEAT-23.SPEC-001 | The screen displays tier, billing cycle, and active-client count against the plan's limit | Phase 2 (Explicit) |
| Upgrade -- subscribe to the paid tier when exceeding the free-tier client count | FEAT-23.SPEC-001, FEAT-23.SPEC-003, FEAT-23.SPEC-004 | The screen offers the Subscribe action and shows the charge outcome; the integration spec submits the charge to the subscription-billing capability; the automation applies the confirmed tier change | Phase 2 (Explicit) |
| Downgrade offer -- reduce plan when client count drops back below the threshold | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-003, FEAT-23.SPEC-004 | The detection automation raises the offer; the screen surfaces it as optional, never forced; accepting submits the downgrade through the integration spec and the state-sync automation applies it | Phase 2 (Explicit) |
| Cancel -- stop the paid plan at any time, effective at the end of the paid period | FEAT-23.SPEC-001, FEAT-23.SPEC-006, FEAT-23.SPEC-003, FEAT-23.SPEC-004 | The screen offers Cancel with a plain end-of-period explanation; the cancel automation hands the request to the integration spec, which relays it to the billing capability; only after the capability's acknowledgment does the cancel automation record status Cancelled while the plan stays Paid and usable, and the state-sync automation applies the tier change once the period actually ends | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-23.SPEC-002 | Free Plan Auto-Provisioning | Phase 3 (Entity-Lifecycle Analysis) | The Create cell of the CRUD matrix for Subscription Plan has no covering spec in the explicit capability list; the dependency map states the entity is "Created by FEAT-23 (free tier, automatically at sign-up)" and the States field confirms "a brand-new account starts on the free tier automatically, no explicit 'no plan' state" -- a genuine, distinct creation trigger from account sign-up (FEAT-20), not from any of the four named capabilities |
| FEAT-23.SPEC-003 | Subscription Billing Processing | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-31) names the subscription-billing capability this feature relies on for charging Nadia's own plan, and the External Touchpoints row marks this capability as "Pending -- assigned when the batch covering these features is validated," expecting this feature to own its Integration spec. Per the standalone-spec decision rule, any trigger-response crossing the product boundary belongs to an Integration spec, never inline in a screen |
| FEAT-23.SPEC-004 | Plan State Sync | Phase 4 (Trigger-Response Analysis) | Every one of the four capabilities changes the Subscription Plan's tier or status only after the subscription-billing capability confirms an outcome (a charge, a renewal, a period-end); this reconciliation step is cross-entity processing logic shared by upgrade, downgrade, and cancellation/lapse, so it is a standalone Automation rather than duplicated inline logic in three different specs |
| FEAT-23.SPEC-007 | Plan Limit & Access Authorization Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits, Access, and States fields together produce 5+ interacting rules (the free-tier client cap gating new/reactivated clients, Nadia-only mutation, Dana's view-only exception, the "does not silently lock the freelancer out of already-active client work mid-session" rule, and the grace/retry window before a failed charge lapses the plan) shared across SPEC-001, SPEC-002, SPEC-004, and cross-feature into FEAT-01 -- past the inline-validation threshold |
| FEAT-23.SPEC-008 | Plan & Billing Notifications | Phase 4 (Notification surfacing) | The Communications field names an email confirmation with a defined audience (Nadia), trigger (plan change, upgrade, failed charge), and content -- not a same-screen toast with no delivery rules, so it needs a standalone Notification spec |

## Entity-Lifecycle Coverage Matrix

**Entity: Subscription Plan**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-23.SPEC-002 | Auto-provisions a Free-tier, Active-status record the instant a new Freelancer Account is created -- no user action, no explicit "no plan" state | Triggered by FEAT-20 (Onboarding / First-Run Setup) account creation |
| Read (single) | FEAT-23.SPEC-001 | Plan & Billing Screen shows tier, active-client count against the limit, and billing cycle | Also read by FEAT-01 (active-client limit check) and FEAT-31 (Dana's status-only view), per the dependency map |
| Read (list) | N/A | The Relationships field sets exactly one Subscription Plan per Freelancer Account (feature-dependency-map.md), so there is never more than one record to list -- no list view is needed | -- |
| Update | FEAT-23.SPEC-003, FEAT-23.SPEC-004 | The integration spec submits upgrade, downgrade, and cancellation requests to the subscription-billing capability; the state-sync automation writes the confirmed tier and status back onto the record | -- |
| Delete/Archive | N/A | This feature has no delete or archive path of its own -- the dependency map states the Subscription Plan is "Deleted by FEAT-24" (account deletion) only, since a subscription cannot outlive the account it belongs to; this is a genuine cross-feature ownership boundary, not an omission | See Data Export & Account Deletion (FEAT-24) |
| State Transition | FEAT-23.SPEC-004 (Free+Active -> Paid+Active on confirmed upgrade; Paid+Active -> Free+Active on an accepted downgrade, immediate on the billing capability's stop-billing confirmation, billing_cycle cleared; Paid+Cancelled -> Free+Active or Free+Lapsed at period end, per active-client count; Paid+Active -> Paid+Charge failed on a failed renewal; Paid+Charge failed -> Paid+Active on retry success or -> Free+Lapsed after the grace window, billing_cycle cleared), FEAT-23.SPEC-005 (flags the Downgrade-eligible condition; no state change), FEAT-23.SPEC-006 (Paid+Active or Paid+Charge failed -> Paid+Cancelled -- ends at period end, recorded only after the billing capability acknowledges the cancellation via FEAT-23.SPEC-003) | The five valid plan states are Free+Active, Paid+Active, Paid+Charge failed, Paid+Cancelled (ends at period end), and Free+Lapsed; "Downgraded" is not a tier or status | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client | FEAT-23.SPEC-001, FEAT-23.SPEC-005 | Reads the freelancer's own active-client count (from FEAT-01) to display usage against the plan's limit and to evaluate the downgrade threshold |
| Freelancer Account | FEAT-23.SPEC-002 | Reads the account-creation event that triggers free-tier auto-provisioning |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| A new Freelancer Account is created | Auto-provision a Free-tier Subscription Plan record | Standalone Automation | SPEC-002 |
| Nadia taps Subscribe (upgrade) | Submit the charge to the subscription-billing capability at the chosen billing cycle | Standalone Integration | SPEC-003 |
| The subscription-billing capability confirms the charge | Set tier to Paid, set billing cycle and status Active, gain the corresponding client capacity | Standalone Automation | SPEC-004 |
| The subscription-billing capability reports a failed charge | Set status to Charge failed with the specific reason; existing client work stays fully accessible; offer immediate retry | Standalone Automation, screen surfaces the reason inline (Error state) | SPEC-004 / SPEC-001 |
| A failed charge is never recovered through the grace/retry window | Transition the plan to Lapsed | Standalone Logic/Rule (grace-window rule), applied by SPEC-004 | SPEC-007 / SPEC-004 |
| Nadia's active-client count drops back below the free-tier threshold while she is on a Paid plan | Detect eligibility and surface a downgrade offer -- never forced | Standalone Automation | SPEC-005 |
| Nadia accepts the downgrade offer | Submit the stop-billing request to the subscription-billing capability, then on its confirmation apply Free+Active immediately (billing_cycle cleared) | Standalone Integration, then Standalone Automation | SPEC-003 / SPEC-004 |
| Nadia declines or ignores the downgrade offer | Remain on the Paid plan; the offer may resurface on a later view | Inline in triggering screen | SPEC-001 |
| Nadia taps Cancel | Relay the cancellation to the billing capability; once it acknowledges, record status Cancelled -- ends at period end; the plan remains Paid and fully usable through the end of the current paid period | Standalone Automation, relayed via Standalone Integration (capability first, then record) | SPEC-006 / SPEC-003 |
| A cancelled plan's paid period ends | Relay the period-end event from the subscription-billing capability, then transition tier to Free (if the active-client count is within the free-tier cap) or to Lapsed (if it is not) | Standalone Integration (inbound event), then Standalone Automation | SPEC-003 / SPEC-004 |
| The plan reaches Lapsed | Block adding or reactivating an active client beyond the free-tier limit until Nadia upgrades again or archives clients; no data is lost and existing client portals stay reachable | Cross-feature -- enforced by FEAT-01's own limit check, which reads this feature's plan status (XBR-23) | FEAT-01 responsibility, reads FEAT-23.SPEC-004's output |
| The plan changes tier (upgrade, downgrade, cancellation confirmed, or lapse) | Send a plan-change confirmation email to Nadia | Standalone Notification | SPEC-008 |
| A subscription charge fails | Send a failed-charge alert email to Nadia | Standalone Notification | SPEC-008 |
| Nadia's active-client count changes (client added, archived, or reactivated) | Re-evaluate the plan's usage-against-limit display and downgrade eligibility | Cross-feature -- triggered by FEAT-01, consumed by SPEC-001 / SPEC-005 | FEAT-01 responsibility |
| A plan-change, upgrade, downgrade, cancellation, or lapse event occurs | Write an append-only Activity Log Entry | Cross-feature -- owned by Immutable Activity & Audit Trail (FEAT-13) | FEAT-13 responsibility |
| Nadia's account is deleted | The Subscription Plan record is deleted as part of account deletion | Cross-feature -- owned by Data Export & Account Deletion (FEAT-24) | FEAT-24 responsibility |

## Shared Context

**Shared Entities:**
- Subscription Plan -- created by SPEC-002, updated by SPEC-003 and SPEC-004 (upgrade/downgrade/cancel/lapse path), state-flagged by SPEC-005 (downgrade eligibility) and SPEC-006 (cancellation), read/displayed by SPEC-001, and governed by SPEC-007 for its limit, access, and grace-window rules. Fields: `tier` (Free, Paid), `billing_cycle` (monthly or yearly, paid only), `active_client_count` (derived from FEAT-01), `status` (Active, Charge failed, Cancelled -- ends at period end, Lapsed). Valid plan states are exactly five: Free+Active, Paid+Active, Paid+Charge failed, Paid+Cancelled -- ends at period end, and Free+Lapsed.
- Client -- read (not owned) by SPEC-001 and SPEC-005 for the freelancer's own active-client count; this feature never writes any Client field.

**Shared UI Patterns:**
- Single-surface plan pattern -- SPEC-001 is the one screen for this feature's entire capability set (view, upgrade, downgrade response, cancel), following the Lifecycle-feature heuristic of a compact, linear surface rather than splitting each action into its own screen; its states (Empty-as-auto-free, Error) are all instances of the same screen rather than distinct specs.
- Never-forced downgrade -- the downgrade offer is always presented as an optional action on SPEC-001, never an automatic plan change, consistently referenced by SPEC-005 (which only flags eligibility) and SPEC-001 (which only offers, never applies, the change).
- No mid-session lockout on billing failure -- a failed charge (SPEC-004) never removes access to already-active client work within the same session; SPEC-001's Error state and SPEC-007's authorization rule both enforce this consistently.

**Shared Validation:**
- SPEC-007 defines the free-tier client cap, the Nadia-only mutation gate, Dana's view-only exception, and the grace/retry window before a failed charge lapses the plan. SPEC-001, SPEC-002, and SPEC-004 all reference SPEC-007 rather than restating these rules, and FEAT-01's own limit-check spec (FEAT-01.SPEC-008) reads the cap value SPEC-007 defines rather than duplicating it.

## Internal Dependency Map

```
SPEC-002 (Free Plan Auto-Provisioning) -> [new Freelancer Account created] -> Subscription Plan record exists (Free, Active)
SPEC-001 (Plan & Billing Screen) -> [Nadia taps Subscribe] -> SPEC-007 (Plan Limit & Access Authorization Rules) -> [pass] -> SPEC-003 (Subscription Billing Processing)
SPEC-003 (Subscription Billing Processing) -> [charge confirmed] -> SPEC-004 (Plan State Sync) -> [tier set to Paid] -> SPEC-001 (screen shows Paid tier and new capacity)
SPEC-003 (Subscription Billing Processing) -> [charge fails] -> SPEC-004 (Plan State Sync) -> [status set to Charge failed] -> SPEC-001 (Error state, reason and retry) / SPEC-008 (failed-charge alert)
SPEC-004 (Plan State Sync) -> [grace window exhausted] -> SPEC-007 (Plan Limit & Access Authorization Rules) -> [lapse rule applies] -> SPEC-004 (status set to Lapsed) -> SPEC-008 (plan-change email)
SPEC-005 (Downgrade Eligibility Detection) -> [active-client count drops below threshold on a Paid plan] -> SPEC-001 (screen surfaces the downgrade offer)
SPEC-001 (Plan & Billing Screen) -> [Nadia accepts the downgrade offer] -> SPEC-003 (Subscription Billing Processing) -> [confirmed] -> SPEC-004 (Plan State Sync) -> [stop-billing confirmed; tier set to Free, status Active, billing_cycle cleared, immediate] -> SPEC-008 (plan-change email)
SPEC-001 (Plan & Billing Screen) -> [Nadia taps Cancel] -> SPEC-006 (Cancel Subscription) -> [hands request to] SPEC-003 (Subscription Billing Processing relays cancellation) -> [billing capability acknowledges; plan not changed before this] -> SPEC-006 (records status Cancelled -- ends at period end, tier stays Paid) -> SPEC-008 (plan-change email)
SPEC-003 (Subscription Billing Processing) -> [period-end event reported] -> SPEC-004 (Plan State Sync) -> [Free+Active if within the free-tier cap, else Free+Lapsed, per active-client count] -> SPEC-001 / SPEC-008
```

**Default Entry:** SPEC-001 (Plan & Billing Screen) -- reached from the Subscription & Account Data area of Settings, from the upgrade prompt raised by Client & Project Management (FEAT-01) when a client is added or reactivated beyond the free-tier limit, or from the storage-limit warning link in Large File Handling & Storage (FEAT-16).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-23.SPEC-002 | Inbound | FEAT-20 (Onboarding / First-Run Setup) | A new Freelancer Account's creation triggers the Free-tier Subscription Plan auto-provisioning | Nadia completes sign-up |
| FEAT-23.SPEC-001, FEAT-23.SPEC-005 | Inbound | FEAT-01 (Client & Project Management) | Reads the freelancer's own active-client count to display usage against the plan's limit and to evaluate downgrade eligibility | A client is added, archived, or reactivated |
| FEAT-23.SPEC-004 | Outbound | FEAT-01 (Client & Project Management) | FEAT-01's own limit-enforcement spec (FEAT-01.SPEC-008) reads this feature's current plan tier and status to gate adding or reactivating an active client beyond the limit, and blocks it while the plan is Lapsed (XBR-23) | Nadia's subscription status changes (upgrade, downgrade, cancel, lapse) |
| FEAT-23.SPEC-001 | Inbound | FEAT-01 (Client & Project Management) | The upgrade prompt Nadia sees originates from FEAT-01's own limit check; this feature's screen is where she completes the subscribe action it opens | Adding or reactivating an active client exceeds the plan's limit |
| FEAT-23.SPEC-001 | Inbound | FEAT-16 (Large File Handling & Storage) | The storage-limit warning links Nadia into this feature's plan view to review or change her plan | Nadia is near her storage allowance |
| FEAT-23.SPEC-001 (plan tier) | Outbound | FEAT-16 (Large File Handling & Storage) | The per-freelancer storage allowance amount is read from this feature's current plan tier | A file is uploaded or the storage usage summary is displayed |
| FEAT-23.SPEC-008 | Outbound | FEAT-14 (Notifications & Email) | The plan-change and failed-charge emails are sent through the transactional email delivery capability owned by FEAT-14; this feature carries no Integration spec of its own for that capability | A plan tier or status change, or a failed charge, occurs |
| FEAT-23.SPEC-001 | Inbound | FEAT-31 (Support Access) | Dana views the plan status read-only inside a logged support session that FEAT-31 owns; she never reaches SPEC-001 itself and can never change the plan | Dana opens a support session |
| FEAT-23.SPEC-002, FEAT-23.SPEC-004 | Inbound | FEAT-24 (Data Export & Account Deletion) | The Subscription Plan record is deleted as part of account deletion; this feature has no delete path of its own | Nadia deletes her account |
| FEAT-23.SPEC-002, FEAT-23.SPEC-004, FEAT-23.SPEC-005, FEAT-23.SPEC-006 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Every plan-change, upgrade, downgrade, cancellation, and lapse event writes an append-only trail entry | Any Subscription Plan status or tier change |

## Non-Functional Notes

**Data volumes / growth:** Exactly one Subscription Plan record per Freelancer Account (Relationships field, feature-dependency-map.md), so volume scales one-to-one with the freelancer roster -- a few thousand freelancers expected in year one (scope-boundaries.md, SC-21). This feature carries no growth concern of its own beyond that ratio. It emits `plan_viewed`, `plan_upgraded`, `plan_downgrade_offered`, `subscription_charge_failed`, `subscription_cancelled`, and `plan_lapsed` signals (product-features.md, Signals field): SPEC-001 fires `plan_viewed`; SPEC-004 fires `plan_upgraded` and `plan_lapsed`; SPEC-005 fires `plan_downgrade_offered`; SPEC-003/SPEC-004 fire `subscription_charge_failed`; SPEC-006 fires `subscription_cancelled`.

**Responsiveness:** This feature is measured directly by the Free-to-Paid Conversion metric -- at least 40% of freelancers who try to add a client beyond the free limit are on a paid plan within 14 days (success-metrics.md, Free-to-Paid Conversion). The upgrade flow (SPEC-001, SPEC-003) shows real progress while the charge is processed rather than a silent wait, and a failed charge is surfaced with its specific reason within the same session, not left to a later email (assumptions-constraints.md, ASMP-27; product-features.md, States field).

**Data sensitivity / privacy:** The Subscription Plan record holds the freelancer's own billing relationship with Clientroom (tier, billing cycle, status); no card or payment data is ever captured or stored by the product, since that handling belongs entirely to the subscription-billing capability (assumptions-constraints.md, ASMP-24; feature-dependency-map.md, Entity: Subscription Plan, Data Sensitivity). Dana's support-session view of this feature is status-only and never exposes billing or payment details (user-persona.md, Access Matrix).

**Compliance flags:** The plan tier, status, and billing cycle are GDPR-class personal data tied to Nadia's own identity, exportable and deletable on request through FEAT-24 (assumptions-constraints.md, ASMP-23, ASMP-24). The failed-charge alert must reach Nadia within minutes rather than being lost (assumptions-constraints.md, ASMP-26), and the Plan & Billing Screen must remain usable with a screen reader and keyboard, keep typed input on errors, and never rely on colour alone to distinguish plan states (assumptions-constraints.md, ASMP-27).

## Non-Goals

- **A time-limited free trial in place of the ongoing free tier** -- Excluded per product-features.md's Rationale ([CHALLENGED] entry, SYN-04 protection): although every profiled competitor uses a trial-only model, the founder's stated pricing model (BRIEF.md, Business Context) is retained deliberately, and the Free-to-Paid Conversion metric exists specifically to test its performance.
- **Per-seat, per-contact, or per-team-member billing** -- Excluded per product-features.md's Rationale ([RESEARCH-INFORMED] entry): pricing is by active-client count only; client contacts are unlimited on every plan. This is reinforced by scope-boundaries.md (SC-01): the product models solo freelancer accounts with no internal-staff seat model, so there is no team-seat dimension to bill against.
- **The platform holding, moving, or processing the freelancer's own card or bank details** -- Excluded per BRIEF.md's Constraints and scope-boundaries.md (SC-10): all charge handling belongs to the subscription-billing capability (SPEC-003); this feature stores only the resulting tier, billing cycle, and status.
- **Client-facing invoicing or payment collection** -- Excluded per product-features.md's Interactions field, which states this feature is "independent of client Invoicing & Payments (FEAT-09/FEAT-10), which is the freelancer's own client's money, never the platform's." This feature governs only Nadia's own subscription to Clientroom.
- **Deleting the Subscription Plan independent of a full account deletion** -- Intentional lifecycle decision surfaced by the CRUD matrix: the record has no delete path of its own (feature-dependency-map.md, Entity: Subscription Plan, Lifecycle: "Deleted by FEAT-24"), because a subscription cannot outlive the account it belongs to.
- **Dana (Support Operator) changing, upgrading, downgrading, or cancelling a freelancer's plan** -- Excluded per scope-boundaries.md (SC-04) and the Access Matrix (user-persona.md): support sessions are read-only in every feature; Dana may view plan status only, inside a logged session, and can never mutate it.
- **A scoped-permission or delegated-billing role for a freelancer's own staff** -- Excluded per scope-boundaries.md (SC-01): the product models solo freelancer accounts only, so only Nadia herself ever views or changes her plan.
