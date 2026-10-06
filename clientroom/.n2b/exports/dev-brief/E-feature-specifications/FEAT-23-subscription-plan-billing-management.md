# FEAT-23 — Subscription Plan & Billing Management

This chapter covers Subscription Plan & Billing Management, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 8 specifications carrying 164 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-23.SPEC-001 | Plan & Billing Screen | screen | 36 |
| FEAT-23.SPEC-002 | Free Plan Auto-Provisioning | automation | 11 |
| FEAT-23.SPEC-003 | Subscription Billing Processing | integration | 22 |
| FEAT-23.SPEC-004 | Plan State Sync | automation | 24 |
| FEAT-23.SPEC-005 | Downgrade Eligibility Detection | automation | 14 |
| FEAT-23.SPEC-006 | Cancel Subscription | automation | 15 |
| FEAT-23.SPEC-007 | Plan Limit & Access Authorization Rules | logic-rule | 24 |
| FEAT-23.SPEC-008 | Plan & Billing Notifications | notification | 18 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Plan & Billing Screen

## Overview

**Name:** Plan & Billing Screen
**ID:** FEAT-23.SPEC-001
**Type:** Screen
**Purpose:** Nadia views her current tier, usage against its client limit, and billing cycle, and initiates upgrade, responds to a downgrade offer, or cancels, from one surface.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Displaying the current plan tier, usage against the client limit, and billing cycle
- Initiating a Subscribe (upgrade) action: billing-cycle selection, the one-time disclosure notice, and the hand-off to and return from the billing capability's payment-entry experience
- Surfacing and responding to an optional downgrade offer (Paid to Free, effective immediately)
- Initiating a Cancel action, with a plain end-of-period explanation in a confirmation dialog
- Showing the Charge failed state inline with its specific reason and an immediate Retry action while retries remain
- Showing a failed Subscribe attempt or rejected downgrade inline without changing the plan

**Non-Goals:**
- Entering or storing payment details -- excluded per BRIEF.md's Constraints and scope-boundaries.md (SC-10): payment details are captured entirely inside the subscription-billing capability's own experience, reached through FEAT-23.SPEC-003, never on this screen.
- The mechanics of submitting a charge, receiving outcomes, or applying a confirmed tier change -- owned by FEAT-23.SPEC-003 (Subscription Billing Processing) and FEAT-23.SPEC-004 (Plan State Sync); this screen only initiates the request and displays the result.
- Choosing which invoicing or payment-collection method a client uses -- entirely separate per product-features.md's Interactions field: this screen governs only Nadia's own subscription to Clientroom, never client-facing Invoicing & Payments (FEAT-09/FEAT-10).
- Changing an already-Paid plan's billing cycle without a tier change (e.g., switching monthly to yearly while staying Paid) -- product-features.md's Key Capabilities name only Upgrade, Downgrade offer, and Cancel; a same-tier cycle switch is not a capability the product definition establishes, so it is not offered here.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia taps the Subscription & Account Data area of Settings | None -- screen loads her current plan |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement) | The "Upgrade" action on the blocked add-client message | None -- screen loads with the Subscribe action already prominent since she is at her limit |
| FEAT-16.SPEC-001 (Storage Usage Summary) | The "Review your plan" link on the storage-limit warning | None -- screen loads her current plan; the storage allowance itself is read from her plan tier by FEAT-16, not carried as a parameter here |
| Email CTA (FEAT-23.SPEC-008) | Nadia taps "View plan," "Upgrade," or "Retry now" in a plan-change or failed-charge email | None beyond routing to this screen; a Retry-now link opens directly onto the Charge failed state's Retry action (or, when retries are used up, onto the retries-used message) |
| Return from the billing capability's payment-entry experience | Nadia completes, cancels, or closes the capability's payment-entry experience and is returned by it | The outcome the capability reports (completed, failed, or abandoned); see Interactions |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Subscribe, accept/decline downgrade offer, Cancel, Retry a failed charge -- all actions on her own plan | -- |
| Dana (Support Operator) | Not applicable to this screen -- her view of plan status is delivered entirely through FEAT-31.SPEC-002 (Operator Support Session Console), a separate read-only mirror; she never opens this screen directly | No | She has no navigation path that reaches this screen; attempting to reach it directly (outside a support session) is treated as any other out-of-scope navigation for her role |
| Owen (Client Primary Contact) | No | No | No settings navigation from the client portal reaches this screen; the client portal product surface contains no Subscription & Account Data area (Access Matrix: None) |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal surface exists for this screen |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on their own dashboard, not this screen, unless they navigated here from a link that re-resolves after authentication |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any in-progress action (e.g., a Subscribe request not yet confirmed) is not assumed to have completed; Nadia re-checks the plan's actual state after signing back in |

## Layout and Content

**Header:** Screen title "Plan & Billing" with a back arrow (returns to Account Profile, FEAT-21.SPEC-001).

**Body, organized top to bottom:**

- **Load-error banner (conditional):** Appears in place of the Current Plan card only when the initial plan load fails. Text "Couldn't load your plan. Try again." with a "Retry" button.
- **Current Plan card:** Shows the tier label ("Free" or "Paid"; a Lapsed plan shows "Free -- Lapsed"), and when Paid, the billing cycle (monthly or yearly) and the next renewal or period-end date. Contains a "How billing works" link at the bottom of the card. Display-only apart from the link.
- **Payment confirmation banner (conditional):** Shown while the screen waits for the capability's outcome after Nadia returns from the payment-entry experience: "Confirming your payment..." with a progress indicator. After 10 seconds it adds "Still confirming -- this is taking longer than usual."
- **Attempt-failed banner (conditional, transient):** Shown after a Subscribe attempt fails or a downgrade request is rejected: "This charge couldn't be completed: {reason}." (Subscribe) or "We couldn't end your billing: {reason}. Your plan is unchanged." (downgrade), with a "Try again" button and a dismiss control. It reflects an attempt, not plan state: it is not stored, and it disappears on reload.
- **Charge failed banner (Error state, conditional):** Appears only when status is Charge failed (a failed renewal charge on a Paid plan). Shows the specific failure reason, and either a "Retry" button (while the grace window is open and retry attempts used are below the retry limit) or, when retries are used up, the message "You've used all your retries. Your plan stays fully usable until {grace_window_end_date}, then it lapses. You can cancel your plan at any time." with no Retry button. Positioned directly below the Current Plan card so it is the first thing Nadia sees if it applies.
- **Downgrade offer banner (conditional):** Appears only when the downgrade-eligible flag is raised (FEAT-23.SPEC-005), which happens only while tier is Paid and status is Active. Shows a plain statement that her client count now fits the free tier, with "Downgrade" and "Keep my Paid plan" actions, both optional. Positioned below the Usage meter.
- **Usage meter:** Shows Nadia's active-client count against her plan's client limit: when Free (Active), the count against the free-tier limit; when Paid (any status), "unlimited"; when Lapsed, the count against the free-tier limit with, if the count exceeds it, the note "Over your free-tier limit -- existing clients stay reachable; adding more is on hold until you upgrade." Display-only.
- **Primary action area:** Shows "Subscribe" when tier is Free and status is Active; shows "Upgrade" when status is Lapsed (the same action as Subscribe); shows "Cancel" when tier is Paid and status is Active or Charge failed; shows nothing (only the plain explanation text) when status is Cancelled -- ends at period end.
- **Billing-cycle selection panel (opens from Subscribe/Upgrade):** Title "Choose your billing cycle." Two mutually exclusive options: "Monthly -- platform parameter: `paid-plan-monthly-price` per month" and "Yearly -- platform parameter: `paid-plan-yearly-price` per year". Neither is preselected. A "Continue" button (disabled until an option is selected) and a "Back" button. If Nadia taps Continue's disabled area or attempts to proceed without a choice, the message "Choose a monthly or yearly billing cycle to continue." appears under the options.
- **Disclosure notice (dialog, shown the first time Nadia proceeds past cycle selection, and on request via "How billing works"):** Text per FEAT-23.SPEC-003 (Consent and Disclosure): "To subscribe, your account reference and the plan you're choosing are shared with our billing partner. Your payment details are entered directly with them and never stored by Clientroom." Buttons "Continue" and "Cancel" when opened as part of Subscribe; a single "Close" button when opened from the "How billing works" link.
- **Downgrade confirmation dialog:** Opens when Nadia taps "Downgrade." Title "Downgrade to Free?" Text: "This takes effect right away and stops your billing -- you won't be charged again, and the rest of your current paid period isn't refunded. You'll have room for {free_tier_client_limit} active clients; your existing clients and portals stay reachable." Buttons "Downgrade now" and "Not now."
- **Cancel confirmation dialog:** Opens when Nadia taps Cancel. Title "Cancel your paid plan?" Text: "Your plan stays active through {period_end_date}. After that, it moves to the free tier or lapses depending on your client count at that time. You can subscribe again at any time." Buttons "Cancel plan" and "Keep my plan."
- **Lapsed explanation (conditional):** Appears only when status is Lapsed. Plain text stating existing clients and portals remain reachable, and an "Upgrade" action to recover capacity.
- **Cancelled explanation (conditional):** Appears only when status is Cancelled -- ends at period end. Plain text stating the exact date the plan remains active through. While a cancellation is being finished (FEAT-23.SPEC-006), this area instead reads "We're finishing your cancellation -- no action needed."

**Footer:** None.

### Responsive Behavior

- **Compact size class:** All cards, banners, and panels stack in a single column, full width, in the order described above; dialogs fill the width with a consistent margin and their two buttons stack vertically.
- **Medium size class and above:** The same single-column stacking is retained, capped at a consistent platform-wide content width and horizontally centered; dialogs are centered at a fixed width; no structural reordering, since this screen has no side-by-side content that benefits from extra width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-21.SPEC-001 (Account Profile) | Screen closes | Standard navigation transition |
| "How billing works" link | Tap | Opens the disclosure notice inline with a single "Close" button | Notice shown over the screen | Notice text; "Close" dismisses it and returns focus to the link |
| Load-error banner -- "Retry" | Tap | Reloads the plan data | Banner shows a progress state | Success: banner is replaced by the Current Plan card and the state that applies. Failure: the banner stays with the same message |
| Subscribe / Upgrade button | Tap | Opens the billing-cycle selection panel | Panel appears; Subscribe/Upgrade is not resubmittable while the flow is open | Panel with Monthly and Yearly options and their prices |
| Billing-cycle option (Monthly or Yearly) | Tap or select by keyboard | Selects that option (one at a time) | Selected option is marked; "Continue" enables | The selected option is visibly and programmatically marked (not by colour alone) |
| Billing-cycle panel -- "Continue" | Tap (enabled only after a selection) | First time ever: opens the disclosure notice. Every later time: proceeds directly to the hand-off | Panel closes or notice opens | See the disclosure and hand-off rows |
| Billing-cycle panel -- "Back" | Tap | Closes the panel; nothing is sent | Panel closes | Screen unchanged; Subscribe available |
| Disclosure notice -- "Continue" (Subscribe flow) | Tap | Proceeds to the hand-off to the billing capability | Notice closes; the screen shows "Taking you to our billing partner..." | Nadia leaves this screen for the capability's payment-entry experience; no data left the product before this tap |
| Disclosure notice -- "Cancel" (Subscribe flow) | Tap | Returns to this screen with nothing sent | Notice and panel close | Screen unchanged; the notice will be shown again next time |
| Disclosure notice -- "Close" (from "How billing works") | Tap | Dismisses the notice | Notice closes | Focus returns to the "How billing works" link |
| Return from payment entry -- completed | The capability returns Nadia to this screen after she completes payment entry | Shows the Payment confirmation banner until FEAT-23.SPEC-003 reports the outcome | Banner "Confirming your payment..." | Success outcome: banner clears; Current Plan card updates to Paid with unlimited capacity; upgrade email arrives separately (FEAT-23.SPEC-008). Failure outcome: the Attempt-failed banner appears with the specific reason and "Try again"; the plan is unchanged |
| Return from payment entry -- abandoned | Nadia closes, backs out of, or cancels the capability's payment-entry experience, and is returned (or later reopens this screen) | Nothing is submitted and no outcome arrives; the plan is not changed | None | If returned by the capability: note "You left before completing payment -- your plan hasn't changed." with Subscribe still available. If she simply closes the tab, the next view shows her plan as it was |
| Attempt-failed banner -- "Try again" | Tap | Reopens the billing-cycle selection panel (Subscribe) or the downgrade confirmation dialog (downgrade) | Banner is dismissed; panel or dialog opens | No limit on attempts; the plan and status are unaffected by failed attempts |
| Attempt-failed banner -- dismiss | Tap | Hides the banner | Banner hidden | None |
| Downgrade offer -- "Downgrade" | Tap | Opens the downgrade confirmation dialog | Dialog appears | Dialog text stating immediate effect, billing stops, no refund of the current period |
| Downgrade confirmation -- "Downgrade now" | Tap | Submits the stop-billing (downgrade) request through FEAT-23.SPEC-003 | Dialog closes; the offer banner shows a progress state | Success: Current Plan card updates to Free (Active) with the free-tier client limit and billing cycle no longer shown; downgrade email arrives separately. If her client count rose above the limit in the meantime, it shows Free -- Lapsed instead. Failure: the Attempt-failed banner shows the specific rejection reason with "Try again"; the plan stays Paid and Active and the offer remains |
| Downgrade confirmation -- "Not now" | Tap | Closes the dialog | Dialog closes | No data change; offer banner still showing |
| Downgrade offer -- "Keep my Paid plan" | Tap | Dismisses the offer for this view only -- no data change | Offer banner is hidden for the remainder of this session | No confirmation needed; the offer may resurface on a later view if eligibility is re-evaluated as raised (FEAT-23.SPEC-005) |
| Cancel button | Tap | Opens the Cancel confirmation dialog | Dialog appears | Dialog text stating the end-of-period effect |
| Cancel dialog -- "Cancel plan" | Tap | Submits the cancellation through FEAT-23.SPEC-006, which waits for the billing capability's acknowledgment before recording it | Dialog closes; Cancel button shows a progress state | Success: screen shows the Cancelled explanation with the exact period-end date; confirmation email arrives separately. Capability slow or down: FEAT-23.SPEC-003's messages ("Still working -- this is taking longer than usual." / "Billing is temporarily unavailable. Try cancelling again in a few minutes."), plan unchanged. Acknowledged but still being recorded: "We're finishing your cancellation -- no action needed." Stale state: screen refreshes to the plan's current actual state with "Your plan status has changed -- here's the latest." |
| Cancel dialog -- "Keep my plan" | Tap | Closes the dialog | Dialog closes | No data change; nothing sent |
| Charge failed banner -- "Retry" | Tap (visible only while the grace window is open and retry attempts used are below the retry limit) | Resubmits the failed renewal charge through FEAT-23.SPEC-003 | Retry button shows a progress state | Success: Charge failed banner clears and Current Plan card confirms Active/Paid (no email). Failure: banner updates with the new specific reason; if retry attempts are now used up, the Retry button is replaced by the retries-used message; otherwise it remains available |
| Lapsed explanation -- "Upgrade" | Tap | Same as the Subscribe button -- opens the billing-cycle selection panel and continues through FEAT-23.SPEC-003 | Same as Subscribe | Same as Subscribe |

### Accessibility Notes

- **Focus order:** Back arrow -> load-error banner and its Retry (when present) -> Current Plan card (read order) including the "How billing works" link -> payment confirmation or attempt-failed banner (when present) -> Charge failed banner and its Retry action or retries-used message (when present) -> Usage meter -> Downgrade offer banner and its two actions (when present) -> primary action (Subscribe, Cancel, or Upgrade) -> conditional explanation text. Dialogs and the billing-cycle panel trap focus while open and return it to the control that opened them.
- **Dynamic announcements:** The Charge failed banner, the retries-used message, the attempt-failed banner, the payment confirmation banner, the downgrade offer banner, and the Cancelled/Lapsed explanation are announced to assistive technology when they first appear on load or after an action completes, since each represents a state change relevant to Nadia's billing standing.
- **Action feedback:** Success and failure outcomes for Subscribe, Downgrade, Cancel, and Retry are announced as they occur; on failure, focus moves to the banner containing the specific reason.
- **Keyboard alternatives:** Every action on this screen (Subscribe, billing-cycle options and buttons, disclosure buttons, Downgrade and its dialog, Keep my Paid plan, Cancel and its dialog, Retry, Try again, Upgrade, How billing works, load-error Retry) is reachable and operable by keyboard; there are no pointer-only gestures. Billing-cycle selection and plan states are never distinguished by colour alone.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Current Plan card and Usage meter show a loading placeholder; no actions are available yet | Screen first opens | Plan data finishes loading |
| Free (default) | Current Plan card shows "Free"; Usage meter shows count against platform parameter: `free-tier-active-client-limit`; Subscribe is the primary action | Plan data loads with tier=Free, status=Active | Nadia subscribes successfully |
| Paid (Active) | Current Plan card shows "Paid" with billing cycle and renewal date; Usage meter shows "unlimited"; Cancel is the primary action | Plan data loads with tier=Paid, status=Active | A cancellation, a failed renewal charge, an accepted downgrade, or (rare) a direct status change occurs |
| Downgrade offered | Same as Paid (Active), plus the downgrade offer banner | The downgrade-eligible flag is raised (FEAT-23.SPEC-005) while tier is Paid and status is Active | Nadia accepts, dismisses (for this session), or the flag clears because her count rose above the threshold or the plan's status changed (e.g., Charge failed or Cancelled) |
| Subscribe in progress | Billing-cycle panel, disclosure notice, or the hand-off message is showing; other Subscribe controls are inactive | Nadia taps Subscribe/Upgrade | She goes Back or Cancel (returns to the prior state), or leaves for the payment-entry experience |
| Confirming payment | Payment confirmation banner with progress indicator; Subscribe and Upgrade are inactive | Nadia is returned from the payment-entry experience after completing it | The outcome arrives: Paid (Active) on success, or the Attempt-failed state on failure |
| Attempt failed (transient) | Attempt-failed banner with the specific reason and "Try again"; the plan card shows the unchanged plan | A Subscribe attempt fails or a downgrade request is rejected | Nadia taps Try again, dismisses the banner, or reloads the screen |
| Charge failed (Error) | Charge failed banner shows the specific reason and Retry; the rest of the screen (Current Plan card, client access) remains fully shown and usable; Cancel remains available | A renewal charge on a Paid plan fails (status Charge failed) | A successful retry (returns to Paid/Active), a cancellation, or the grace window ends (moves to Lapsed) |
| Charge failed -- retries used | Same as Charge failed, but the Retry button is replaced by "You've used all your retries. Your plan stays fully usable until {grace_window_end_date}, then it lapses. You can cancel your plan at any time." | Charge failed while retry attempts used equal platform parameter: `subscription-charge-retry-count` and the grace window is still open | The grace window ends (Lapsed) or Nadia cancels |
| Cancelled (pending period end) | Current Plan card still shows "Paid"; the Cancelled explanation text shows the exact period-end date; no Subscribe/Cancel primary action is offered (already cancelled) | Nadia's cancellation is acknowledged and recorded (FEAT-23.SPEC-006) | The period-end event arrives and the plan transitions to Free or Lapsed (FEAT-23.SPEC-004) |
| Lapsed | Current Plan card shows "Free -- Lapsed"; Usage meter shows the count against the free-tier limit (with the over-limit note if applicable); the Lapsed explanation states existing clients remain reachable; Upgrade is the primary action | Grace window exhausted, or period end reached with active-client count over the free-tier limit | Nadia upgrades successfully |
| Error (load failure) | Load-error banner "Couldn't load your plan. Try again." with a Retry button in place of the Current Plan card | The initial plan data load fails | Nadia taps Retry and the load succeeds |
| Offline/Degraded | Banner "You're offline -- plan and billing changes need a connection. You can still view your last-loaded plan details." at the top; Subscribe, billing-cycle Continue, Downgrade, Cancel, and Retry controls are disabled; previously loaded plan data remains visible | Connectivity lost while the screen is open, or the screen is opened without connectivity after a prior successful load | Connectivity is restored -- controls re-enable and the plan is re-fetched to confirm it reflects the latest state |

## Validation Rules

Validation and authorization for every action on this screen are governed by FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules). See that spec for the free-tier threshold, the grace/retry window and retry limit, and the exact conditions under which Subscribe, downgrade acceptance, Cancel, and Retry are available. This screen applies those conditions by showing or hiding each action per the current plan state. The one input this screen validates itself is the billing cycle: exactly one of Monthly or Yearly must be selected before "Continue" proceeds, otherwise "Choose a monthly or yearly billing cycle to continue."

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-21.SPEC-001 (Account Profile) | FEAT-21 |
| Subscribe / Upgrade action, billing-cycle selection, and disclosure notice | Stay on this screen until the disclosure notice's "Continue" (or, on later subscriptions, the panel's "Continue") | -- |
| Disclosure notice "Continue" / panel "Continue" (after the first time) | The subscription-billing capability's own payment-entry experience (through FEAT-23.SPEC-003); the capability returns Nadia to this screen with the outcome, or she abandons it and the plan is unchanged | Subscription-billing capability (external) |
| Successful Cancel | Stays on this screen, now showing the Cancelled explanation | -- |
| "How billing works" link | Opens the disclosure notice inline; no navigation away from this screen | -- |

## Data Model

**Creates:** None.
**Reads:** Subscription Plan -- tier, billing_cycle, active_client_count, status, downgrade-eligible flag, last failure reason and retry attempts used (when Charge failed), grace-window end date (when Charge failed), period-end date (when Cancelled or Paid).
**Updates:** None directly -- all writes happen through FEAT-23.SPEC-003 (billing submission), FEAT-23.SPEC-004 (state sync), FEAT-23.SPEC-005 (flag), and FEAT-23.SPEC-006 (cancellation); this screen only initiates those requests.
**Deletes:** None.

## Business Rules

- The downgrade offer is always optional -- accepting or declining is entirely Nadia's choice; the screen never applies a downgrade automatically (Shared UI Pattern, Feature Breakdown Brief; FEAT-23.SPEC-005). The offer exists only while tier is Paid and status is Active (FEAT-23.SPEC-007); it is not shown for Charge failed, Cancelled, or Lapsed. A downgrade is Paid to Free, effective immediately, with billing stopped and no refund of the remaining period.
- A Charge failed status never removes access to already-active client work shown or reachable from elsewhere in the product during the same session (Shared UI Pattern, Feature Breakdown Brief; FEAT-23.SPEC-007).
- Only a failed renewal charge on a Paid plan produces the Charge failed state. A failed Subscribe attempt or a rejected downgrade is an inline, transient failure that leaves the plan exactly as it was (FEAT-23.SPEC-004).
- XBR-23: while the plan is Lapsed (always tier Free), the primary action offered here is Upgrade, since adding or reactivating clients beyond the free-tier limit requires an active paid plan again; Subscribe is authorized on Lapsed by FEAT-23.SPEC-007.
- All actions on this screen are governed by FEAT-23.SPEC-007's Authorization Rules -- only Nadia can act, and only under the conditions that spec defines for each action.

## Edge Cases

- **Nadia navigates away mid-Subscribe with the billing-cycle panel or disclosure notice open but not yet confirmed** -- No request has been submitted yet, so nothing changes; returning to this screen later shows her prior tier unchanged and Subscribe available to try again.
- **Nadia taps Subscribe twice rapidly** -- The second tap is ignored while the first request is in progress (button in a progress state, or the panel already open).
- **Nadia abandons the billing capability's payment-entry experience** -- Nothing is submitted, the plan is unchanged, and (if the capability returns her here) she sees "You left before completing payment -- your plan hasn't changed." with Subscribe available. No Charge failed state and no email result.
- **Nadia completes payment entry but the outcome takes long to arrive** -- The Confirming payment banner stays, adding "Still confirming -- this is taking longer than usual." after 10 seconds; the plan changes only when the outcome arrives, and if she leaves and returns, the screen shows whatever the plan's actual state is by then.
- **Network failure while submitting a Subscribe, downgrade, or Cancel request** -- Banner: "Couldn't complete this action. Check your connection and try again." with a Retry option; the plan's state is unchanged since no confirmation was received.
- **Nadia's plan state changed since this screen last loaded (e.g., another session already cancelled, or a charge failed) while she attempts an action here** -- The action is rejected as stale (reject-with-refresh, per the dependency map's Contention note for Subscription Plan) and the screen refreshes to show "Your plan status has changed -- here's the latest," reflecting the actual current state.
- **Concurrent-edit conflict: Nadia has this screen open in two tabs and cancels in one while attempting to accept a downgrade offer in the other** -- The second action (downgrade accept) is rejected as stale once the cancellation has committed, since a Cancelled -- ends at period end plan is no longer eligible for a downgrade offer; the second tab refreshes to show the Cancelled state. Resolution: reject-with-refresh, consistent with the dependency map's Contention note that billing-capability status reports (and the most recently committed local action) are authoritative over a concurrent viewer's stale load.
- **Nadia loses connectivity mid-session with a Charge failed banner already showing** -- The banner and its reason remain visible (last-loaded data), but Retry is disabled with the Offline/Degraded banner until connectivity returns.
- **The last retry attempt fails while the grace window is still open** -- The Retry button is replaced by the retries-used message with the window's end date; Cancel remains available; the plan lapses at the window's end, not before.
- **A renewal charge fails while the downgrade offer is showing** -- On next load or refresh the offer is gone (the offer needs status Active) and the Charge failed banner is shown; after a successful retry the offer may reappear if the count is still within the free-tier limit.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-003 (Subscription Billing Processing) | Triggers (outbound) | Subscribe, downgrade-accept, Retry, and Cancel actions submit requests through this spec; its disclosure and degradation messages surface here |
| FEAT-23.SPEC-004 (Plan State Sync) | References (inbound) | Supplies the tier/status this screen displays after every outcome, and the reasons for failed subscribe attempts and rejected downgrades |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | References (inbound) | Supplies the downgrade-eligible flag this screen surfaces as the offer |
| FEAT-23.SPEC-006 (Cancel Subscription) | Triggers (outbound) | The confirmed Cancel action initiates this automation |
| FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules) | References (inbound) | Governs every action's availability and conditions on this screen |
| FEAT-23.SPEC-008 (Plan & Billing Notifications) | References (outbound) | Every action this screen completes triggers the corresponding confirmation or alert email |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound/outbound) | Entry point from, and back-arrow return to, Settings |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement) | Navigation (inbound) | The blocked add-client message's "Upgrade" action opens this screen |
| FEAT-16.SPEC-001 (Storage Usage Summary) | Navigation (inbound) | The storage-limit warning's "Review your plan" link opens this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| plan_viewed | tier, status | Screen finishes loading | supports success-metrics.md: "Free-to-Paid Conversion" |
| plan_upgrade_initiated | from_tier, billing_cycle_selected | Nadia taps "Continue" on the billing-cycle panel (and, the first time, on the disclosure notice) and is handed to the payment-entry experience | supports success-metrics.md: "Free-to-Paid Conversion" |
| plan_downgrade_offer_responded | response: accepted / kept_paid | Nadia taps "Downgrade now" in the confirmation dialog or "Keep my Paid plan" on the offer banner | supports success-metrics.md: "Free-to-Paid Conversion" |
| plan_cancel_initiated | from_tier, billing_cycle | Nadia taps "Cancel plan" in the Cancel confirmation dialog | supports success-metrics.md: "Free-to-Paid Conversion" |
| charge_retry_initiated | prior_failure_reason, retry_attempts_used | Nadia taps Retry on the Charge failed banner | N/A -- no distinct Stage 2 metric tracks retry attempts specifically; retained alongside FEAT-23.SPEC-003's subscription_charge_failed event so the retry behavior this feature promises is observable |
| payment_entry_abandoned | billing_cycle_selected | Nadia is returned from the payment-entry experience without completing it | N/A -- no Stage 2 metric measures abandonment inside the billing capability's experience; retained so drop-off between "Continue" and a confirmed charge is observable alongside plan_upgrade_initiated |

## Acceptance Criteria

**FEAT-23.SPEC-001-AC-01:** Given Nadia's plan is Free with active_client_count within the limit, when she opens this screen, then she sees the Free tier, her usage against platform parameter: `free-tier-active-client-limit`, and a Subscribe action.

**FEAT-23.SPEC-001-AC-02:** Given Nadia is at her free-tier client limit and arrives via the FEAT-01.SPEC-008 upgrade prompt, when this screen loads, then Subscribe is immediately visible as the primary action.

**FEAT-23.SPEC-001-AC-03:** Given Nadia is on the Free tier, when she taps Subscribe, selects a billing cycle, accepts the disclosure notice (first time), completes payment entry with the billing capability, and the charge succeeds, then the Current Plan card updates to show Paid with unlimited client capacity.

**FEAT-23.SPEC-001-AC-04:** Given Nadia's downgrade-eligible flag is raised (tier Paid, status Active), when this screen loads, then the downgrade offer banner appears with both "Downgrade" and "Keep my Paid plan" options.

**FEAT-23.SPEC-001-AC-05:** Given the downgrade offer is showing, when Nadia taps "Keep my Paid plan," then the offer is hidden for this session and no plan data changes.

**FEAT-23.SPEC-001-AC-06:** Given the downgrade offer is showing, when Nadia taps "Downgrade," reads the dialog stating the change is immediate, stops billing, and does not refund the current period, taps "Downgrade now," and the billing capability confirms billing has stopped, then the Current Plan card updates to Free with the billing cycle no longer shown and no charge made.

**FEAT-23.SPEC-001-AC-07:** Given Nadia's plan is Paid and Active, when she taps Cancel, confirms "Cancel plan," and the billing capability acknowledges, then the screen shows the Cancelled explanation with the exact period-end date.

**FEAT-23.SPEC-001-AC-08:** Given Nadia's plan status is Charge failed, when this screen loads, then the Charge failed banner shows the specific reason and a Retry action (retries remaining), and her already-active client work elsewhere remains unaffected.

**FEAT-23.SPEC-001-AC-09:** Given Nadia's plan status is Charge failed, when she taps Retry and the charge succeeds, then the banner clears and the Current Plan card confirms Active/Paid.

**FEAT-23.SPEC-001-AC-10:** Given Nadia's plan status is Charge failed with retries remaining, when she taps Retry and the charge fails again, then the banner updates with the new specific reason and Retry remains available within the grace window until the retry limit is reached.

**FEAT-23.SPEC-001-AC-11:** Given Nadia's plan status is Lapsed, when this screen loads, then the Current Plan card shows "Free -- Lapsed," she sees the Lapsed explanation confirming existing clients remain reachable, an Upgrade action, and a usage meter showing her count against the free-tier limit (not "unlimited").

**FEAT-23.SPEC-001-AC-12:** Given the initial plan data load fails, when the screen opens, then a "Couldn't load your plan. Try again." banner appears with a Retry option.

**FEAT-23.SPEC-001-AC-13:** Given Nadia loses connectivity while viewing this screen, when connectivity drops, then the offline banner appears, all mutating controls disable, and her last-loaded plan details remain visible.

**FEAT-23.SPEC-001-AC-14:** Given Nadia regains connectivity after being offline on this screen, when connectivity is restored, then controls re-enable and the plan is re-fetched.

**FEAT-23.SPEC-001-AC-15:** Given Nadia taps Subscribe twice rapidly, when the second tap occurs while the first is in progress, then the second tap is ignored.

**FEAT-23.SPEC-001-AC-16:** Given a network failure occurs while Nadia submits Cancel, when the failure is detected, then she sees "Couldn't complete this action. Check your connection and try again." with Retry, and her plan is unchanged.

**FEAT-23.SPEC-001-AC-17:** Given Nadia's plan state changed in another session since this screen last loaded, when she attempts an action here, then it is rejected as stale and the screen refreshes to show "Your plan status has changed -- here's the latest."

**FEAT-23.SPEC-001-AC-18:** Given Nadia has this screen open in two tabs and cancels in one, when she then attempts to accept a downgrade offer in the other tab, then that attempt is rejected as stale and the second tab refreshes to the Cancelled state.

**FEAT-23.SPEC-001-AC-19:** Given Dana has no open support session, when she attempts to reach this screen directly, then no such navigation path exists for her role.

**FEAT-23.SPEC-001-AC-20:** Given Owen is signed in to his client portal, when he looks for any navigation to this screen, then none exists.

**FEAT-23.SPEC-001-AC-21:** Given an unauthenticated visitor attempts to open this screen's link, when the attempt is made, then they are redirected to sign-in.

**FEAT-23.SPEC-001-AC-22:** Given Nadia's session expires while she has an in-progress Subscribe request not yet confirmed, when she signs back in, then the screen re-checks and displays the plan's actual current state rather than assuming the request completed.

**FEAT-23.SPEC-001-AC-23:** Given Nadia tapped Subscribe and the billing-cycle panel is open with neither option selected, when she views the panel, then "Continue" is disabled, and if she tries to proceed the message "Choose a monthly or yearly billing cycle to continue." appears; once she selects Monthly or Yearly, "Continue" enables and shows the price per platform parameter: `paid-plan-monthly-price` or platform parameter: `paid-plan-yearly-price`.

**FEAT-23.SPEC-001-AC-24:** Given Nadia is subscribing for the first time and has selected a cycle, when she taps "Continue," then the disclosure notice appears with "Continue" and "Cancel"; when she taps "Cancel," she returns to this screen with nothing sent and the plan unchanged; when she taps "Continue," she is taken to the billing capability's payment-entry experience.

**FEAT-23.SPEC-001-AC-25:** Given Nadia has already seen the disclosure notice, when she taps "How billing works," then the notice reopens with a single "Close" button, no data is sent, and "Close" returns focus to the link.

**FEAT-23.SPEC-001-AC-26:** Given Nadia taps Cancel, when the confirmation dialog opens, then it states her plan stays active through {period_end_date} and offers "Cancel plan" and "Keep my plan"; tapping "Keep my plan" closes the dialog with nothing sent and no data changed.

**FEAT-23.SPEC-001-AC-27:** Given Nadia completes payment entry and is returned to this screen, when the capability's outcome has not yet arrived, then the "Confirming your payment..." banner shows (adding "Still confirming -- this is taking longer than usual." after 10 seconds) and the plan card does not change until the outcome arrives.

**FEAT-23.SPEC-001-AC-28:** Given Nadia abandons the billing capability's payment-entry experience and is returned to this screen, when the screen loads, then she sees "You left before completing payment -- your plan hasn't changed." with Subscribe available, and no Charge failed state is shown.

**FEAT-23.SPEC-001-AC-29:** Given Nadia is on the Free tier and her Subscribe attempt fails, when the failure is reported, then the Attempt-failed banner shows "This charge couldn't be completed: {reason}." with "Try again," her plan stays Free with its prior status, no Charge failed banner appears, and reloading the screen clears the banner.

**FEAT-23.SPEC-001-AC-30:** Given Nadia's status is Charge failed and her retry attempts used equal platform parameter: `subscription-charge-retry-count` with the grace window still open, when this screen loads, then the Retry button is replaced by "You've used all your retries. Your plan stays fully usable until {grace_window_end_date}, then it lapses. You can cancel your plan at any time." and Cancel remains available.

**FEAT-23.SPEC-001-AC-31:** Given the load-error banner is showing, when Nadia taps Retry and the plan loads, then the banner is replaced by the Current Plan card and the state that applies to her plan.

**FEAT-23.SPEC-001-AC-32:** Given Nadia taps "Downgrade now" and the billing capability rejects the request, when the rejection is reported, then the Attempt-failed banner shows "We couldn't end your billing: {reason}. Your plan is unchanged." with "Try again," and the plan stays Paid and Active with the offer still available.

**FEAT-23.SPEC-001-AC-33:** Given Nadia's plan status is Charge failed and her active-client count is within the free-tier limit, when this screen loads, then no downgrade offer banner is shown; given a successful retry then returns status to Active, then the offer banner appears.

**FEAT-23.SPEC-001-AC-34:** Given Nadia is Lapsed and taps Upgrade, when she selects a cycle, continues, and the charge succeeds, then the Current Plan card updates to Paid with unlimited capacity and the Lapsed explanation is gone.

**FEAT-23.SPEC-001-AC-35:** Given Nadia's cancellation was acknowledged but the record is still being written, when this screen loads, then the Cancelled explanation area reads "We're finishing your cancellation -- no action needed." until the record completes.

**FEAT-23.SPEC-001-AC-36:** Given Nadia's plan is Paid and Nadia accepts a downgrade while her live client count has risen above the free-tier limit before the capability confirms, when the confirmation is applied, then the Current Plan card shows "Free -- Lapsed" with the over-limit note on the usage meter.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 23 | 23 |
| States | 13 (loading, free, paid, downgrade offered, subscribe in progress, confirming payment, attempt failed, charge failed, charge failed with retries used, cancelled, lapsed, load error, offline) | 13 |
| Business Rules | 5 | 5 |
| Edge Cases | 10 | 10 |



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



# Integration Spec: Subscription Billing Processing

## Overview

**Name:** Subscription Billing Processing
**ID:** FEAT-23.SPEC-003
**Type:** Integration
**Purpose:** Submits Nadia's own subscribe, stop-billing (downgrade), retry, and cancellation requests to the subscription-billing capability and receives back charge outcomes, renewal and period-end events, and failure reasons.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Submitting a charge when Nadia subscribes (upgrade) and resubmitting a failed renewal charge when she taps Retry
- Requesting that billing stop immediately when Nadia accepts a downgrade offer (a downgrade is Paid to Free with no charge), and when a plan lapses after its grace window
- Relaying a cancellation request and waiting for the capability's acknowledgment, so the paid period continues to its natural end
- Receiving subscribe, downgrade, cancellation-acknowledged, renewal, retry, and period-end outcomes and handing their consequences to FEAT-23.SPEC-004 (or, for cancellation, FEAT-23.SPEC-006)
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to Nadia about what billing data is shared with the capability

**Non-Goals:**
- Choosing the subscription-billing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for this capability.
- Applying the confirmed tier or status change to the Subscription Plan record -- owned by FEAT-23.SPEC-004 (Plan State Sync), which this spec's inbound events route to.
- Holding, moving, or storing Nadia's own card or bank details -- excluded per BRIEF.md's Constraints and scope-boundaries.md (SC-10): those details are captured and held entirely inside the subscription-billing capability's own experience, never inside the product.
- Client-facing payment collection (invoices, deposits paid by Owen) -- entirely separate from this spec, which concerns only Nadia's own subscription to Clientroom; that capability is described by FEAT-10.SPEC-003 and connected through FEAT-32.

## Capability Category

**Category:** Subscription billing
**Dependency Source:** ASMP-31 -- "Subscription-billing capability for the freelancer's own plan" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Subscription billing for the freelancer's own plan" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-23)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Nadia subscribes to the paid tier and gains client capacity once the charge is confirmed | Upgrade -- subscribe to the paid tier when exceeding the free-tier client count | FEAT-23.SPEC-001 (Plan & Billing Screen), FEAT-23.SPEC-004 (Plan State Sync) |
| Nadia accepts a downgrade offer and her plan goes from Paid to Free, effective immediately, with billing stopped and no charge | Downgrade offer -- reduce plan when client count drops back below the threshold | FEAT-23.SPEC-001, FEAT-23.SPEC-004 |
| Nadia cancels her plan and it stays Paid and usable through the end of the current paid period, shown as cancelled only once the capability has acknowledged it | Cancel -- stop the paid plan at any time, effective at the end of the paid period | FEAT-23.SPEC-006 (Cancel Subscription), FEAT-23.SPEC-004 |
| Nadia's plan renews automatically each billing cycle without her having to re-subscribe | View current plan -- see tier and usage against its client limit (renewal keeps the tier current) | FEAT-23.SPEC-001, FEAT-23.SPEC-004 |
| A failed renewal charge is surfaced to Nadia with its specific reason and an immediate retry, without cutting off her already-active client work; a failed subscribe attempt is surfaced inline and changes nothing | Upgrade (failure path of the same capability) | FEAT-23.SPEC-001, FEAT-23.SPEC-004, FEAT-23.SPEC-008 (Plan & Billing Notifications) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Freelancer Account reference | Freelancer Account -- account reference only | Nadia subscribes, retries a charge, accepts a downgrade, or cancels; or a plan lapses after its grace window | Ties the billing relationship to the correct freelancer |
| Requested billing cycle | Subscription Plan -- billing_cycle (Nadia's selection) | Nadia subscribes | The capability must know whether to charge monthly or yearly |
| Subscribe request | Subscription Plan -- requested tier (Paid) | Nadia subscribes | Tells the capability what plan to charge going forward |
| Charge retry request | Subscription Plan -- reference to the failed renewal charge | Nadia taps Retry on the Charge failed banner | Asks the capability to attempt the failed charge again |
| Stop-billing request | Subscription Plan -- stop-billing-now flag | Nadia accepts a downgrade offer; or FEAT-23.SPEC-004 lapses a plan after the grace window | Tells the capability to end billing immediately and issue no further charges; carries no charge and no refund or proration request |
| Cancellation request | Subscription Plan -- cancellation flag | Nadia confirms Cancel (FEAT-23.SPEC-006) | Tells the capability to stop renewing at the end of the current paid period |

No card number, bank account detail, or any other payment credential ever leaves the product, because the product never captures them -- Nadia enters payment details directly inside the subscription-billing capability's own experience (ASMP-24, scope-boundaries.md SC-10).

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Subscribe charge succeeded outcome | A subscribe charge completes successfully | Subscription Plan -- tier, billing_cycle, status (via FEAT-23.SPEC-004) |
| Subscribe charge failed outcome with a plain-language failure reason | A subscribe charge does not complete | Nothing persisted on the plan; the reason is shown inline by FEAT-23.SPEC-001 (via FEAT-23.SPEC-004, which makes no write) |
| Stop-billing confirmed / rejected outcome | The capability confirms it has stopped billing, or rejects the request (with a plain-language reason) | Confirmed: Subscription Plan -- tier, billing_cycle, status (via FEAT-23.SPEC-004). Rejected: nothing persisted; the reason is shown inline |
| Renewal charge failed outcome with a plain-language failure reason | A renewal charge on an existing Paid plan does not complete, or a retry of it does not complete | Subscription Plan -- status, last failure reason, first-failure timestamp, retry attempts used (via FEAT-23.SPEC-004) |
| Renewal confirmed | A billing cycle renews successfully on an existing Paid plan | Subscription Plan -- status remains Active, billing period advances (via FEAT-23.SPEC-004) |
| Retry succeeded outcome | Nadia's retry of a failed renewal charge completes | Subscription Plan -- status returns to Active, failure record cleared, billing period advances (via FEAT-23.SPEC-004) |
| Cancellation acknowledged, with the current billing period's end date | The capability confirms it will not renew | Handed to FEAT-23.SPEC-006, which records status Cancelled -- ends at period end |
| Period-end event | A cancelled plan's current paid period ends | Subscription Plan -- tier, billing_cycle, and status transition (via FEAT-23.SPEC-004) |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Subscribe charge succeeded | Nadia's subscribe charge completes | Handed to FEAT-23.SPEC-004, which sets tier=Paid, billing_cycle as selected, status=Active | Plan screen shows the Paid tier and new client capacity; upgrade confirmation email sent | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Subscribe charge failed | A subscribe charge does not complete | Handed to FEAT-23.SPEC-004, which makes no write: tier, status, and billing_cycle stay as they were, no grace window starts | Plan screen shows "This charge couldn't be completed: {reason}." inline with "Try again"; no email | FEAT-23.SPEC-004, FEAT-23.SPEC-001 |
| Stop-billing confirmed (downgrade accepted) | The capability confirms billing has stopped after Nadia accepted the offer | Handed to FEAT-23.SPEC-004, which sets tier=Free, clears billing_cycle, and sets status=Active (or Lapsed if the live client count is over the free-tier limit) | Plan screen shows the Free tier; downgrade (or lapse) email sent | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Stop-billing rejected (downgrade accepted) | The capability rejects the request | Handed to FEAT-23.SPEC-004, which makes no write; the plan stays Paid and Active | Plan screen shows the reason inline with the offer still available; no email | FEAT-23.SPEC-004, FEAT-23.SPEC-001 |
| Renewal charge failed | An existing Paid plan's renewal charge does not complete, or a retry of it does not complete | Handed to FEAT-23.SPEC-004, which sets status=Charge failed with the specific reason (first failure) or increments retry attempts used (failed retry) | Plan screen shows the failure reason inline with an immediate Retry option (until retries are used up); failed-charge alert email sent on the first failure only | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Retry succeeded | Nadia's retry of a failed renewal charge completes | Handed to FEAT-23.SPEC-004, which returns status to Active and clears the failure record | Charge failed banner clears; no email (the pending failed-charge alert, if unsent, is cancelled) | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Renewal succeeded | An existing Paid plan's billing cycle renews on schedule | Handed to FEAT-23.SPEC-004, which keeps status=Active and advances the billing period | No user-facing interruption -- the plan simply continues; this is a silent success per the product's "no confirmation for routine renewal" design decision | FEAT-23.SPEC-004 |
| Cancellation acknowledged | The capability confirms Nadia's cancellation request | Handed to FEAT-23.SPEC-006, which sets status=Cancelled -- ends at period end | Plan screen shows the Cancelled explanation with the exact period-end date; cancellation confirmation email sent | FEAT-23.SPEC-006, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Period-end reached on a cancelled plan | The paid period Nadia's cancellation was scheduled against ends | Handed to FEAT-23.SPEC-004, which sets tier=Free and clears billing_cycle, with status Active (if active_client_count is within the free-tier limit) or Lapsed (if not) | Plan screen reflects the new tier/status on next view; paid-plan-ended email (Free) or lapse email (Lapsed) sent | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Stop-billing acknowledged after a lapse | The capability confirms it has stopped billing a plan that lapsed after its grace window | None -- the plan is already Free and Lapsed | None | FEAT-23.SPEC-004 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-23.SPEC-001 (Plan & Billing Screen) -- Subscribe action | The Subscribe button shows a progress state; after 10 seconds a note appears: "Still working -- this is taking longer than usual." The rest of the screen remains fully usable. | The button is disabled with "Billing is temporarily unavailable. Your current plan is unaffected -- try again in a few minutes." Nadia's existing client work is unaffected. | The screen shows the specific rejection reason in plain language: "This charge couldn't be completed: {reason}. Try again or use a different payment method inside your billing details." This is a failed attempt, not a status change: the plan is unchanged (tier, status, and billing_cycle as before), there is no Charge failed status, no grace window, and no email, and Nadia may try again without limit. |
| FEAT-23.SPEC-001 -- Accept downgrade offer (stop-billing request) | Same progress-then-note pattern as Subscribe. | Same disabled-with-message pattern as Subscribe; the downgrade offer remains available to accept once billing is back. | Reason shown inline: "We couldn't end your billing: {reason}. Your plan is unchanged -- try again." Nadia stays Paid and Active, the offer remains available, and no email is sent. |
| FEAT-23.SPEC-001 -- Cancel action | Progress state; after 10 seconds "Still working -- this is taking longer than usual." The plan is not shown as Cancelled until the capability acknowledges. | Cancel is disabled with "Billing is temporarily unavailable. Try cancelling again in a few minutes." Nadia's plan is not marked Cancelled and continues unaffected -- because a cancellation is recorded only after the capability acknowledges it, the plan can never show Cancelled while the capability is still set to renew. | N/A -- a cancellation request has no rejection path from the capability's side; a request that receives no acknowledgment is treated as the slow or down behavior above and leaves the plan unchanged. |
| FEAT-23.SPEC-001 -- Charge failed banner Retry | Retry shows a progress state; after 10 seconds the same "Still working" note appears. The attempt is counted against the retry limit only once an outcome is received. | Retry is disabled with "Billing is temporarily unavailable. Your plan stays usable until {grace_window_end_date} -- try again in a few minutes." The grace window is not extended for downtime. | The banner updates with the new specific reason; the attempt counts against platform parameter: `subscription-charge-retry-count` (FEAT-23.SPEC-007); when retries are used up the Retry control is replaced by the retries-used message. |

## Consent and Disclosure

- **First subscribe disclosure** -- The first time Nadia proceeds with Subscribe, after she has chosen a billing cycle, a notice appears before she is routed into the subscription-billing capability's own payment-entry experience: "To subscribe, your account reference and the plan you're choosing are shared with our billing partner. Your payment details are entered directly with them and never stored by Clientroom." Options: "Continue" and "Cancel." "Continue" routes her to the capability's payment-entry experience; "Cancel" returns her to the Plan & Billing Screen with nothing sent. Shown once; afterward, a "How billing works" link on the Plan & Billing Screen reopens the same notice for reading (with a single "Close" button, since no action is pending).
- **Downgrade and cancellation disclosure** -- Accepting a downgrade or cancelling reuses the same disclosure notice, since the same account reference and plan-change request are shared each time; Nadia is not asked to re-consent to a notice she has already seen, but the "How billing works" link remains available at every step.
- **What is never shared** -- No card number, bank detail, invoice content, client data, or any Subscription Plan field beyond the tier/billing-cycle/cancellation/stop-billing request itself ever leaves the product. This boundary is stated in the disclosure notice.

## Edge Cases

- **A second subscribe-charge-succeeded event arrives for a plan that already shows Paid (double submission or another session)** -- The second event changes nothing: the plan already reflects Paid with the confirmed billing cycle, and no duplicate confirmation email fires (FEAT-23.SPEC-008's deduplication rule).
- **The same charge-succeeded event is delivered twice** -- The second delivery changes nothing: a plan already set to Paid with the confirmed billing cycle stays as it is, and no duplicate confirmation email fires.
- **Events arrive out of order (renewal-succeeded before a still-pending renewal-charge-failed for the same cycle)** -- The plan reflects the most recent event by event time, not arrival time; an out-of-order renewal-succeeded that predates a charge-failed event is superseded once the charge-failed event's timestamp is recognized as later.
- **Period-end event arrives for an account that has since been deleted (FEAT-24)** -- The event is discarded with no effect and no user feedback fires, since the Subscription Plan record itself was already deleted as part of account deletion (feature-dependency-map.md, Entity: Subscription Plan, Lifecycle).
- **Capability goes down mid-Subscribe, after the disclosure notice was accepted but before a charge outcome is confirmed** -- The Plan & Billing Screen shows the capability-down message and the plan stays on its prior tier -- no half-upgraded state; Nadia can retry once billing is available again.
- **Nadia abandons the capability's payment-entry experience without completing it** -- No outcome is reported, so nothing changes; FEAT-23.SPEC-001 shows her plan unchanged with Subscribe available (its own return/abandon handling).
- **The capability acknowledges a cancellation but the local record cannot be written afterward** -- Owned by FEAT-23.SPEC-006 (the acknowledgment is already in hand; the record write retries there); if the capability's period-end event arrives first, FEAT-23.SPEC-004 applies it authoritatively.
- **The capability is unavailable when a stop-billing request must be relayed after a lapse** -- The plan is already Free and Lapsed and stays usable; the request is retried at platform parameter: `stop-billing-relay-retry-interval` until acknowledged, with no user-facing state. A renewal-succeeded event that lands on the Lapsed plan in the meantime is discarded by FEAT-23.SPEC-004 with no state change; refunding any such stray charge is handled by the billing partner's own process and is out of scope for this feature.
- **Nadia accepts a downgrade while a renewal charge is in flight** -- The stop-billing request and the renewal are ordered by event time; if the renewal completes first, it is applied then the downgrade proceeds against the current plan; if the stop-billing confirmation is first, the later renewal event is discarded for a Free plan.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Triggered by (inbound) | Subscribe, accept-downgrade, Cancel, and Retry actions initiate requests through this spec |
| FEAT-23.SPEC-001 | Affects (outbound) | Degradation states, disclosure notices, and attempt-failed reasons surface here |
| FEAT-23.SPEC-004 (Plan State Sync) | Triggers (outbound) and Triggered by (inbound) | Every inbound event hands its consequence to this automation; this automation hands back stop-billing requests after a lapse |
| FEAT-23.SPEC-006 (Cancel Subscription) | Triggered by (inbound) and Triggers (outbound) | A cancellation request is relayed to the capability through this spec, and the capability's acknowledgment is handed back for recording |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | References (inbound) | An accepted downgrade offer's stop-billing request flows through this spec |
| FEAT-23.SPEC-008 (Plan & Billing Notifications) | Triggers (outbound) | Subscribe-succeeded, stop-billing-confirmed, renewal-charge-failed, cancellation-acknowledged, and period-end events lead to plan-change and failed-charge emails |
| FEAT-24 (Data Export & Account Deletion) | References (inbound) | Account deletion removes the Subscription Plan; later period-end events for a deleted account are discarded |

## Analytics and Success Signals

- **subscription_charge_submitted** (action: upgrade / retry; billing_cycle) -- supports success-metrics.md: "Free-to-Paid Conversion"
- **subscription_charge_succeeded** (action: upgrade / renewal / retry) -- supports success-metrics.md: "Free-to-Paid Conversion"
- **subscription_charge_failed** (action: subscribe / renewal / retry, reason) -- N/A -- no Stage 2 metric measures charge failure frequency directly; product-features.md's Signals field names this event to keep failure visibility measurable, so it is retained here as the source of that signal for FEAT-23.SPEC-004 and FEAT-23.SPEC-008.
- **subscription_cancellation_relayed** () -- supports success-metrics.md: "Free-to-Paid Conversion" (cancellations are the inverse signal the conversion metric's 14-day window is measured against)
- **subscription_billing_stopped** (trigger: downgrade / lapse, outcome: confirmed / rejected) -- N/A -- no Stage 2 metric measures billing-stop outcomes; retained so the downgrade and lapse paths are observable.
- **billing_degradation_shown** (condition: slow / down / rejected) -- N/A -- no Stage 2 metric measures degradation frequency; retained so the product's tolerance for capability trouble is observable.

## Acceptance Criteria

**FEAT-23.SPEC-003-AC-01:** Given Nadia is on the free tier at her client limit, when she taps Subscribe, selects a billing cycle, accepts the disclosure notice, and the capability confirms the charge, then her Subscription Plan is handed to FEAT-23.SPEC-004 with tier=Paid and the selected billing_cycle.

**FEAT-23.SPEC-003-AC-02:** Given Nadia has accepted a downgrade offer, when the capability confirms it has stopped billing, then the confirmation is handed to FEAT-23.SPEC-004 as a Paid-to-Free downgrade, no charge was submitted, and billing_cycle is cleared by that spec.

**FEAT-23.SPEC-003-AC-03:** Given Nadia confirms Cancel, when the cancellation request is submitted, then it is relayed to the capability so the current paid period continues to its scheduled end, and the capability's acknowledgment is handed to FEAT-23.SPEC-006.

**FEAT-23.SPEC-003-AC-04:** Given an existing Paid plan's billing cycle renews successfully, when the renewal-succeeded event arrives, then the plan's status remains Active with no user-facing interruption.

**FEAT-23.SPEC-003-AC-05:** Given a cancelled plan's paid period ends and Nadia's active-client count is within the free-tier limit, when the period-end event arrives, then FEAT-23.SPEC-004 transitions tier to Free (billing_cycle cleared, status Active).

**FEAT-23.SPEC-003-AC-06:** Given a cancelled plan's paid period ends and Nadia's active-client count exceeds the free-tier limit, when the period-end event arrives, then FEAT-23.SPEC-004 sets tier to Free and status to Lapsed instead of Active.

**FEAT-23.SPEC-003-AC-07:** Given a renewal charge fails, when the charge-failed event arrives with a specific reason, then it is handed to FEAT-23.SPEC-004 for the Charge failed status and to FEAT-23.SPEC-008 for the failure alert email.

**FEAT-23.SPEC-003-AC-08:** Given Nadia taps Subscribe while the capability is slow, when 10 seconds pass without a response, then the note "Still working -- this is taking longer than usual." appears and the rest of the screen stays usable.

**FEAT-23.SPEC-003-AC-09:** Given Nadia taps Subscribe while the capability is down, when the request cannot be sent, then Subscribe is disabled with "Billing is temporarily unavailable. Your current plan is unaffected -- try again in a few minutes."

**FEAT-23.SPEC-003-AC-10:** Given Nadia taps Subscribe and the capability rejects the charge, when the rejection is received, then she sees "This charge couldn't be completed: {reason}. Try again or use a different payment method inside your billing details.", her plan is unchanged, no Charge failed status is set, and no email is sent.

**FEAT-23.SPEC-003-AC-11:** Given Nadia taps Cancel while the capability is down, when the request cannot be sent, then Cancel is disabled with "Billing is temporarily unavailable. Try cancelling again in a few minutes." and her plan is not marked Cancelled and continues unaffected.

**FEAT-23.SPEC-003-AC-12:** Given Nadia has never subscribed before, when she has chosen a billing cycle and proceeds, then the disclosure notice appears with "Continue" and "Cancel," and no data leaves the product until she chooses "Continue."

**FEAT-23.SPEC-003-AC-13:** Given a charge-succeeded event was already applied, when the same event is delivered a second time, then nothing changes and no duplicate confirmation email fires.

**FEAT-23.SPEC-003-AC-14:** Given a period-end event arrives for an account already deleted through FEAT-24, when the event is processed, then it is discarded with no effect and no user feedback.

**FEAT-23.SPEC-003-AC-15:** Given the capability goes down after Nadia accepts the disclosure notice but before a charge outcome is confirmed, when she reopens her plan, then it shows her prior tier unchanged -- no half-upgraded state.

**FEAT-23.SPEC-003-AC-16:** Given Nadia accepts a downgrade offer and the capability rejects the stop-billing request, when the rejection is received, then she sees the reason inline, her plan stays Paid and Active, the offer remains available, and no email is sent.

**FEAT-23.SPEC-003-AC-17:** Given Nadia's status is Charge failed, when she taps Retry and the capability confirms the charge, then the retry-succeeded outcome is handed to FEAT-23.SPEC-004 and no email is sent for it.

**FEAT-23.SPEC-003-AC-18:** Given Nadia's status is Charge failed, when she taps Retry and the capability reports a new failure, then the new reason is handed to FEAT-23.SPEC-004, counted as one retry attempt, and no additional failed-charge alert email fires.

**FEAT-23.SPEC-003-AC-19:** Given Nadia's status is Charge failed and the capability is down, when she views the banner, then Retry is disabled with "Billing is temporarily unavailable. Your plan stays usable until {grace_window_end_date} -- try again in a few minutes." and the grace window is not extended.

**FEAT-23.SPEC-003-AC-20:** Given a plan lapses after its grace window, when FEAT-23.SPEC-004 hands a stop-billing request to this spec and the capability is unavailable, then the request is retried at platform parameter: `stop-billing-relay-retry-interval` until acknowledged and Nadia sees no additional state or message.

**FEAT-23.SPEC-003-AC-21:** Given Nadia abandons the capability's payment-entry experience without completing it, when she returns to the Plan & Billing Screen, then her plan is unchanged and no outcome event was handed to FEAT-23.SPEC-004.

**FEAT-23.SPEC-003-AC-22:** Given a Subscribe attempt fails for a plan that is Free with status Lapsed, when the failure is handed to FEAT-23.SPEC-004, then the plan stays Free and Lapsed with no grace window, and Nadia may try again.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 5 | 5 |
| Inbound Events | 10 | 10 |
| Degradation Paths | 11 (4 screens x conditions, 1 N/A excluded) | 11 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 9 | 9 |



# Automation Spec: Plan State Sync

## Overview

**Name:** Plan State Sync
**ID:** FEAT-23.SPEC-004
**Type:** Automation
**Purpose:** Applies the subscription-billing capability's reported outcomes and period-end/renewal events to the Subscription Plan record's tier and status.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Reconciling subscribe-succeeded, downgrade-confirmed, renewal-failed, renewal-succeeded, retry-succeeded, and period-end events (from FEAT-23.SPEC-003) into the Subscription Plan's tier, billing_cycle, and status
- Counting failed renewal retries and clearing the failure record when a retry succeeds
- Applying the grace/retry window rule (FEAT-23.SPEC-007) that lapses a plan whose failed charge is never recovered
- Recording every resulting tier or status change for FEAT-23.SPEC-008's confirmation and failure emails, for the Activity & Audit Trail (FEAT-13.SPEC-003), and for FEAT-23.SPEC-005's re-evaluation
- Discarding billing events for a plan removed by account deletion (FEAT-24.SPEC-004)

**Non-Goals:**
- Submitting the original charge, downgrade, or cancellation request -- owned by FEAT-23.SPEC-003 (Subscription Billing Processing); this automation only reconciles what that spec's inbound events report.
- Deciding whether Nadia is downgrade-eligible -- owned by FEAT-23.SPEC-005 (Downgrade Eligibility Detection); this automation only applies a tier change once Nadia has accepted an offer and the charge is confirmed.
- Recording the cancellation request itself -- owned by FEAT-23.SPEC-006 (Cancel Subscription); this automation applies the resulting tier/status change only once the period-end event arrives.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Subscribe charge succeeded | FEAT-23.SPEC-003 (Subscription Billing Processing) | Fires when the capability confirms a subscribe charge (from tier Free, status Active or Lapsed) | Plan reference, confirmed tier (Paid), selected billing_cycle, event timestamp |
| Subscribe attempt failed | FEAT-23.SPEC-003 | Fires when a subscribe charge does not complete | Plan reference, specific failure reason, event timestamp |
| Downgrade confirmed (billing stopped) | FEAT-23.SPEC-003 | Fires when the capability confirms it has stopped billing after Nadia accepted the downgrade offer -- no charge is involved | Plan reference, event timestamp |
| Downgrade rejected | FEAT-23.SPEC-003 | Fires when the capability rejects the stop-billing request | Plan reference, specific rejection reason, event timestamp |
| Renewal charge failed | FEAT-23.SPEC-003 | Fires when an existing Paid plan's renewal charge does not complete, and again for each failed retry of that charge | Plan reference, specific failure reason, attempt timestamp |
| Renewal succeeded | FEAT-23.SPEC-003 | Fires when an existing Paid plan's billing cycle renews on schedule | Plan reference, new billing-period boundaries, event timestamp |
| Retry succeeded | FEAT-23.SPEC-003 | Fires when Nadia's retry of a failed renewal charge is confirmed | Plan reference, new billing-period boundaries, event timestamp |
| Period-end reached | FEAT-23.SPEC-003 | Fires when the capability reports the paid period has ended for a plan that was cancelled | Plan reference, current active_client_count (read live from FEAT-01) |
| Grace window exhausted | FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules) | Fires when a plan has held status Charge failed for platform parameter: `subscription-charge-grace-window-days` with no successful retry | Plan reference, first-failure timestamp |
| Account deletion hold or completion | FEAT-24.SPEC-004 (Account Deletion Processing) | Fires when the account's entities are marked pending-delete (hold phase) and again when the Subscription Plan is hard-deleted | Freelancer Account reference, phase (hold / deleted) |

## Processing Logic

1. Receive the reported event and the plan reference it concerns. Compare the event's own timestamp with the timestamp of the last event applied to this plan; an event older than the last applied one is discarded as stale (event time, not arrival time, decides).
2. If the plan no longer exists (account deletion completed, FEAT-24.SPEC-004), discard the event with no effect. If the plan is pending-delete (hold phase), apply the event normally so a restored account is current, but send no email (FEAT-23.SPEC-008 cancels pending deliveries at completion).
3. For **subscribe charge succeeded**: set tier to Paid, billing_cycle to the confirmed selection, status to Active; clear any failure record. Steps 12-13 follow.
4. For **subscribe attempt failed**: make no write to the plan. Tier, status, and billing_cycle stay exactly as they were (Free with Active or Lapsed), no grace window starts, no lapse can follow, and the failed-charge alert email does not fire. Pass the specific reason to FEAT-23.SPEC-001, which shows it inline to Nadia on the Subscribe flow.
5. For **downgrade confirmed**: read the live active_client_count. If it is within platform parameter: `free-tier-active-client-limit`, set tier to Free and status to Active; if it exceeds the limit (clients were added after Nadia accepted), set tier to Free and status to Lapsed. In both cases clear billing_cycle in the same write and clear any failure record. The downgrade takes effect immediately, is not a charge, and carries no proration or refund. Steps 12-13 follow.
6. For **downgrade rejected**: make no write. The plan stays Paid and Active; pass the specific reason to FEAT-23.SPEC-001 with the offer left available.
7. For **renewal charge failed**: if status is Active, set status to Charge failed, record the specific reason, the failure timestamp (which starts the grace window), and retry attempts used as 0; leave tier and billing_cycle unchanged (the plan stays Paid and fully usable). If status is already Charge failed (this is a failed retry), add 1 to retry attempts used and replace the recorded reason, keeping the original first-failure timestamp; no additional alert fires. If status is Cancelled -- ends at period end, or the plan is Free, discard the event (no renewal is expected). Record a first failure for the failed-charge alert (FEAT-23.SPEC-008).
8. For **renewal succeeded**: if status is Active, keep status and tier, advance the billing-period boundaries, and send no email. If status is Charge failed, process it exactly as **retry succeeded** (step 9). If the plan is Cancelled -- ends at period end (the capability renews nothing after acknowledging cancellation) or is Free (including Lapsed, where a charge landed before billing was stopped), discard the event with no state change.
9. For **retry succeeded**: set status from Charge failed to Active, clear the first-failure timestamp, recorded reason, and retry attempts used (this ends the grace window), and advance the billing-period boundaries. A successful retry counts as the renewal for that cycle, so no second renewal event is expected, but it is not announced: no plan-change email fires and any failed-charge alert still pending delivery is cancelled (FEAT-23.SPEC-008). Tier and billing_cycle are unchanged.
10. For **period-end reached**: applies to a plan with status Cancelled -- ends at period end; if the capability reports a period end for a Paid plan that still shows Active or Charge failed (the cancellation was acknowledged by the capability but never recorded locally, FEAT-23.SPEC-006), apply it the same way because the capability's report is authoritative. Read the live active_client_count. If it is within the limit, set tier to Free, status to Active; if it exceeds the limit, set tier to Free, status to Lapsed. In both cases clear billing_cycle in the same write. Steps 12-13 follow.
11. For **grace window exhausted**: set tier to Free, status to Lapsed, clear billing_cycle in the same write, and clear the failure record. Hand a stop-billing request to FEAT-23.SPEC-003 so the capability does not keep charging a plan that is now Lapsed (retried per that spec until acknowledged).
12. After every tier or status write (steps 3, 5, 7, 9, 10, 11): notify FEAT-23.SPEC-005 of the change so it re-evaluates the downgrade offer; record the change for FEAT-23.SPEC-008 (steps 3, 5, 7-first-failure, 10, 11 only; never for retry attempts, retry-succeeded, or routine renewal); and report the change as a record-worthy event (event type, prior and new tier/status, timestamp, actor "Automatic" or Nadia where her action caused it) to FEAT-13.SPEC-003 (Activity Entry Recording) for the append-only trail. The trail write is FEAT-13's responsibility and retries there until it succeeds; it never reverses or delays a plan change that the billing capability has already made authoritative.
13. In every case, confirm the write completes before returning control, so no screen or automation reading the plan ever observes an intermediate, undefined state.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Upgrade applied | Subscribe charge succeeded | tier=Paid, billing_cycle set, status=Active | Plan & Billing Screen shows the Paid tier and new client capacity; upgrade email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Subscribe attempt failed (no change) | Subscribe charge does not complete | None -- tier, status, billing_cycle unchanged | Plan & Billing Screen shows the reason inline with a "Try again" action; no email | FEAT-23.SPEC-001 |
| Downgrade applied | Downgrade confirmed, live active_client_count within the free-tier limit | tier=Free, status=Active, billing_cycle cleared; billing stopped immediately, no charge | Plan & Billing Screen shows the Free tier; downgrade email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Downgrade applied over the limit | Downgrade confirmed, live active_client_count exceeds the limit | tier=Free, status=Lapsed, billing_cycle cleared | Plan & Billing Screen shows the Lapsed state; lapse email sent (in place of the downgrade email) | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003, FEAT-01.SPEC-008 |
| Downgrade rejected (no change) | Capability rejects the stop-billing request | None -- plan stays Paid, Active | Plan & Billing Screen shows the rejection reason with the offer still available; no email | FEAT-23.SPEC-001 |
| Charge-failed status set | First failed renewal charge on a Paid, Active plan | status=Charge failed, reason, first-failure timestamp, retry attempts used=0; tier and billing_cycle unchanged | Plan & Billing Screen shows the reason inline with a Retry action; failed-charge alert email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Retry failed | Failed retry while status is Charge failed | retry attempts used +1, reason replaced; status, tier, timestamp unchanged | Plan & Billing Screen banner shows the new reason; once retry attempts used equal platform parameter: `subscription-charge-retry-count` the Retry control is replaced by the retries-used message; no email | FEAT-23.SPEC-001 |
| Charge recovered | Retry succeeded (or renewal succeeded while Charge failed) | status=Active; failure record cleared; billing-period boundaries advance | Charge failed banner clears; no email; pending failed-charge alert cancelled | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Renewal recorded (no visible change) | Renewal succeeded while Active | Billing-period boundaries advance; status/tier unchanged | None -- routine renewal is silent by design | -- |
| Plan freed at period end | Period-end event, live active_client_count within the free-tier limit | tier=Free, status=Active, billing_cycle cleared | Plan & Billing Screen shows the Free tier on next view; paid-plan-ended email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Plan lapsed at period end | Period-end event, live active_client_count exceeds the limit | tier=Free, status=Lapsed, billing_cycle cleared | Plan & Billing Screen shows the Lapsed state with an explanation that existing clients remain reachable but growth is blocked; lapse email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003, FEAT-01.SPEC-008 |
| Plan lapsed after grace window | Grace window exhausted with no successful retry | tier=Free, status=Lapsed, billing_cycle cleared, failure record cleared; stop-billing request handed to FEAT-23.SPEC-003 | Same as above | FEAT-23.SPEC-001, FEAT-23.SPEC-003, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003, FEAT-01.SPEC-008 |
| Event discarded | Event is stale, or the plan was deleted by account deletion, or (renewal event) status is Cancelled/Free | None | None | -- |
| Sync failure | The reconciliation write itself cannot complete | No change is applied -- the prior state is preserved rather than left half-written | Plan & Billing Screen continues showing the last known state with no false confirmation; the automation retries automatically | FEAT-23.SPEC-001 |

## Data Model

**Reads:** Subscription Plan -- current tier, status, billing_cycle, failure record, active_client_count (the last read live from Client & Project Management, FEAT-01, at downgrade and period-end reconciliation).
**Creates:** None.
**Updates:** Subscription Plan -- tier, billing_cycle, status, failure record (reason, first-failure timestamp, retry attempts used), billing-period boundaries, per the outcome applied.
**Deletes:** None.

## Business Rules

- XBR-23: Lapsing never removes existing data or client-portal reachability -- it only blocks adding or reactivating clients beyond the free-tier limit until Nadia upgrades again or archives clients enough to fit within it. A Lapsed plan is always tier Free with billing_cycle cleared (FEAT-23.SPEC-007), so Subscribe/Upgrade is available to recover.
- The Charge failed status never triggers a mid-session lockout: already-active client work stays fully accessible for the entire grace window (Shared UI Pattern, Feature Breakdown Brief; FEAT-23.SPEC-007).
- Only a failed renewal charge on a Paid plan produces Charge failed and a grace window. A failed subscribe attempt and a rejected downgrade leave the plan untouched (no status change, no grace window, no alert email).
- A failed renewal charge does not immediately demote the plan's tier -- it only sets status to Charge failed and starts the grace window (FEAT-23.SPEC-007); only exhausting the grace window without a successful retry produces Lapsed. Exhausting the retry count does not lapse the plan early; the window's end does.
- A downgrade is Paid to Free, effective immediately when the capability confirms billing has stopped; it is not a charge and no proration or refund is calculated.
- The grace window's threshold values (platform parameter: `subscription-charge-retry-count`, platform parameter: `subscription-charge-grace-window-days`) are owned by FEAT-23.SPEC-007; this automation applies them without redefining them.
- Routine renewal produces no confirmation email or in-app notice, and neither does a recovered charge -- only a change to tier or status other than recovery (upgrade, downgrade, plan ended at period end, or lapse) triggers a plan-change email, per product-features.md's Communications field and FEAT-23.SPEC-008.

## Edge Cases

- **A charge-failed event and a successful retry arrive in quick succession** -- The most recent event by event time wins: if the retry's success timestamp is later than the failure, status returns to Active immediately and the grace window is cleared; a failure timestamp that arrives after an already-applied success is discarded as stale.
- **The grace window closes at the exact moment a retry succeeds** -- The retry is evaluated against the grace-window boundary as inclusive of its end moment (FEAT-23.SPEC-007); a retry confirmed at or before that instant is applied as a success, and the automatic lapse does not fire.
- **Retry count is used up while the window is still open** -- The plan stays Charge failed and fully usable; the Retry control is replaced by the retries-used message; the lapse happens at the window's end.
- **Period-end reconciliation runs while Nadia is actively adding a client in another session** -- The client-add (FEAT-01.SPEC-008) re-checks the plan's current state at the moment of its own commit (reject-with-refresh, per the dependency map's Contention note); whichever of the two operations commits first is authoritative for the other's next check.
- **Concurrent trigger firing (a renewal-succeeded and a charge-failed event for the same plan at effectively the same time)** -- These two outcomes are mutually exclusive by event time: the later event, by its own reported timestamp, is authoritative and the earlier one is treated as superseded (the same event-time-not-arrival-time rule FEAT-23.SPEC-003 applies).
- **Trigger fires while a previous run is in flight for the same plan** -- A second sync for the same Subscription Plan record cannot begin until the first commits, since there is exactly one plan per account and only one billing relationship generates events for it; syncs for different freelancers' plans proceed independently and never queue behind each other.
- **Sync write itself fails after an event is received** -- The plan's prior state is preserved (not left half-applied), the automation retries automatically, and no confirmation email fires until the write actually succeeds -- an email must never announce a change that did not take effect.
- **Account deletion (FEAT-24.SPEC-004) reaches the hold phase or completes while an event is in flight** -- During the hold phase the event is applied normally (the hold is reversible) but emits no email; once the Subscription Plan is hard-deleted, the in-flight run and any later billing event for that plan are discarded with no effect and no feedback.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-003 (Subscription Billing Processing) | Triggered by (inbound) and Triggers (outbound) | Every inbound billing event hands its consequence here; this automation hands back a stop-billing request after a grace-window lapse |
| FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules) | Triggered by (inbound) | The grace-window-exhausted condition fires this automation |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Affects (outbound) | Displays the resulting tier and status, and the subscribe-failed and downgrade-rejected reasons |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | Triggers (outbound) | Every tier or status write notifies it to re-evaluate the downgrade offer |
| FEAT-23.SPEC-006 (Cancel Subscription) | References (inbound) | Cancellation recorded there produces the period-end event this automation later applies |
| FEAT-23.SPEC-008 (Plan & Billing Notifications) | Triggers (outbound) | Every tier/status change (except routine renewal and recovery) and every first failed renewal charge fires a plan-change or failed-charge email |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement, FEAT-01) | Affects (outbound) | Reads the resulting plan status and tier to gate client capacity (XBR-23) |
| FEAT-13.SPEC-003 (Activity Entry Recording, FEAT-13) | Triggers (outbound) | Every committed tier/status change is reported as an append-only trail entry (Brief Cross-Feature Touchpoint); FEAT-13.SPEC-003's own trigger list is owned by FEAT-13 |
| FEAT-24.SPEC-004 (Account Deletion Processing, FEAT-24) | Triggered by (inbound) | Account deletion marks the plan pending-delete, then hard-deletes it; billing events for a deleted plan are discarded |

## Analytics and Success Signals

- **plan_upgraded** (from_tier, to_tier: paid, billing_cycle) -- supports success-metrics.md: "Free-to-Paid Conversion"
- **plan_downgraded** (from_tier, to_tier: free, resulting_status) -- N/A -- no Stage 2 metric measures downgrade volume directly; product-features.md's Signals field names plan_downgrade_offered (emitted by FEAT-23.SPEC-005), not the applied downgrade, so this event is retained for completeness of the tier-change record without a Stage 2 citation.
- **subscription_charge_failed** (action: subscribe / renewal / renewal_retry, reason) -- N/A -- no Stage 2 metric measures charge-failure frequency; product-features.md's Signals field names this event to keep the failure path observable, matching FEAT-23.SPEC-003's own citation of the same gap.
- **charge_recovered** (retry_attempts_used) -- N/A -- no Stage 2 metric measures recovery of failed renewal charges; retained so the effectiveness of the retry window is observable.
- **plan_lapsed** (reason: grace_window_exhausted / period_end_over_limit / downgrade_over_limit) -- supports success-metrics.md: "Free-to-Paid Conversion" (a lapse after a failed charge or an over-limit period end is the negative outcome the conversion metric's 14-day window is measured against)

## Acceptance Criteria

**FEAT-23.SPEC-004-AC-01:** Given Nadia's subscribe charge is confirmed by FEAT-23.SPEC-003, when this automation processes the event, then tier is set to Paid, billing_cycle is set to her selection, and status is set to Active.

**FEAT-23.SPEC-004-AC-02:** Given Nadia has accepted a downgrade offer and the capability confirms billing has stopped, when this automation processes the event with her live active_client_count within the free-tier limit, then tier is set to Free, status is set to Active, billing_cycle is cleared, and no charge was made.

**FEAT-23.SPEC-004-AC-03:** Given a Paid, Active plan's renewal charge fails, when this automation processes the event, then status is set to Charge failed with the specific reason, first-failure timestamp, and retry attempts used of 0 recorded, and tier and billing_cycle are left unchanged.

**FEAT-23.SPEC-004-AC-04:** Given an existing Paid plan's billing cycle renews successfully, when this automation processes the renewal event, then status remains Active with no confirmation email sent.

**FEAT-23.SPEC-004-AC-05:** Given a cancelled plan's paid period ends and Nadia's active-client count is within the free-tier limit, when this automation processes the period-end event, then tier is set to Free, status is set to Active, and billing_cycle is cleared.

**FEAT-23.SPEC-004-AC-06:** Given a cancelled plan's paid period ends and Nadia's active-client count exceeds the free-tier limit, when this automation processes the period-end event, then tier is set to Free, status is set to Lapsed, and billing_cycle is cleared.

**FEAT-23.SPEC-004-AC-07:** Given a plan has held Charge failed status for platform parameter: `subscription-charge-grace-window-days` with no successful retry, when this automation processes the grace-window-exhausted condition, then tier is set to Free, status is set to Lapsed, billing_cycle is cleared, and a stop-billing request is handed to FEAT-23.SPEC-003.

**FEAT-23.SPEC-004-AC-08:** Given Nadia's plan is Charge failed within the grace window, when she continues using her already-active client work, then nothing about that work is blocked.

**FEAT-23.SPEC-004-AC-09:** Given a plan just transitioned to Lapsed, when Nadia's existing clients are viewed, then no data is lost and every existing client portal remains reachable.

**FEAT-23.SPEC-004-AC-10:** Given a renewal-charge-failed event and a later successful-retry event both arrive for the same plan, when this automation reconciles them by event time, then the later success wins and status returns to Active.

**FEAT-23.SPEC-004-AC-11:** Given a retry succeeds at the exact instant the grace window would otherwise close, when this automation evaluates the boundary, then the retry is honored as a success and no lapse occurs.

**FEAT-23.SPEC-004-AC-12:** Given the sync write for a confirmed event fails, when this automation detects the failure, then the plan's prior state is preserved, no confirmation email fires, and the automation retries automatically.

**FEAT-23.SPEC-004-AC-13:** Given a renewal-succeeded and a charge-failed event for the same plan arrive at effectively the same time, when this automation reconciles them, then the event with the later reported timestamp is authoritative.

**FEAT-23.SPEC-004-AC-14:** Given two different freelancers' plans each receive an event at effectively the same time, when this automation processes both, then each plan is reconciled independently with no interference between them.

**FEAT-23.SPEC-004-AC-15:** Given Nadia's plan is Free (Active or Lapsed) and her subscribe charge fails, when this automation processes the event, then tier, status, and billing_cycle are unchanged, no grace window starts, no failed-charge alert email is sent, and the reason is passed to FEAT-23.SPEC-001 to show inline.

**FEAT-23.SPEC-004-AC-16:** Given Nadia's downgrade request is rejected by the billing capability, when this automation processes the event, then the plan stays Paid and Active, no email is sent, and the reason is passed to FEAT-23.SPEC-001.

**FEAT-23.SPEC-004-AC-17:** Given Nadia's status is Charge failed and her retry succeeds, when this automation processes the retry-succeeded event, then status is set to Active, the first-failure timestamp, reason, and retry attempts used are cleared, billing-period boundaries advance, no plan-change email is sent, and any failed-charge alert still pending delivery is cancelled.

**FEAT-23.SPEC-004-AC-18:** Given Nadia's status is Charge failed and a retry fails, when this automation processes the renewal-charge-failed event, then retry attempts used increase by 1, the reason is replaced, the original first-failure timestamp is kept, and no additional alert email is sent.

**FEAT-23.SPEC-004-AC-19:** Given Nadia's live active_client_count exceeds the free-tier limit at the moment a confirmed downgrade is processed, when this automation applies it, then tier is set to Free, status is set to Lapsed, billing_cycle is cleared, and the lapse email is sent instead of the downgrade email.

**FEAT-23.SPEC-004-AC-20:** Given the capability reports a period end for a Paid plan whose local status still shows Active because the cancellation was acknowledged but not recorded, when this automation processes the event, then it applies the period-end outcome exactly as for a Cancelled plan.

**FEAT-23.SPEC-004-AC-21:** Given this automation commits any tier or status change other than a routine renewal, when the write completes, then it notifies FEAT-23.SPEC-005 and reports the change to FEAT-13.SPEC-003 as an append-only trail event, and a trail-write retry never reverses the plan change.

**FEAT-23.SPEC-004-AC-22:** Given Nadia's account has been hard-deleted through FEAT-24.SPEC-004, when a billing event for her former plan arrives, then it is discarded with no effect and no feedback.

**FEAT-23.SPEC-004-AC-23:** Given Nadia's account is in the pending-delete hold phase, when a billing event arrives for her plan, then it is applied to the plan and no email is sent.

**FEAT-23.SPEC-004-AC-24:** Given Nadia's status is Charge failed and her retry attempts used equal platform parameter: `subscription-charge-retry-count` with the grace window still open, when this automation evaluates the plan, then status stays Charge failed, the Retry control is replaced by the retries-used message on FEAT-23.SPEC-001, and the lapse occurs only when the window ends.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 10 | 10 |
| Outcome Paths | 14 | 14 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |



# Automation Spec: Downgrade Eligibility Detection

## Overview

**Name:** Downgrade Eligibility Detection
**ID:** FEAT-23.SPEC-005
**Type:** Automation
**Purpose:** Evaluates the active-client count against the free-tier threshold whenever the count or the plan's tier or status changes, raises the downgrade offer on a Paid, Active plan that sits at or below the threshold, and clears it the moment the plan stops being eligible.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Re-evaluating downgrade eligibility every time Nadia's active-client count changes, and every time the plan's tier or status changes
- Raising a downgrade-eligible flag that FEAT-23.SPEC-001 surfaces as an optional offer, only while tier is Paid and status is Active
- Clearing the flag whenever the plan becomes ineligible: the client count rises above the threshold, or the plan's status or tier changes so that it no longer qualifies
- Reporting a raised offer to the Activity & Audit Trail (FEAT-13.SPEC-003)

**Non-Goals:**
- Applying the downgrade -- owned by FEAT-23.SPEC-003 (Subscription Billing Processing) and FEAT-23.SPEC-004 (Plan State Sync); this automation only flags eligibility, never changes the plan itself.
- Forcing a downgrade -- excluded per product-features.md's Primary Flows: "she is offered, not forced, a downgrade"; this automation never removes Nadia's paid tier on its own.
- Evaluating eligibility for the free-tier client cap itself (blocking a new client) -- owned by FEAT-01.SPEC-008 (Active Client Limit Enforcement) and FEAT-23.SPEC-007; this automation only concerns the downgrade offer on an already-Paid plan.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Active-client count changes (client added, archived, or reactivated) | FEAT-01 (Client & Project Management) -- cross-feature event, not owned by a single FEAT-01 spec ID per the Brief's Side-Effect Inventory | Fires on every change to Nadia's Active-client count, regardless of which FEAT-01 action caused it | Freelancer's plan reference, new active_client_count value |
| Plan tier or status changes (upgrade, downgrade, charge failed, charge recovered, lapse, period end) | FEAT-23.SPEC-004 (Plan State Sync) | Fires after every committed tier or status write | Plan reference, new tier, new status; active_client_count is read live from FEAT-01 |
| Cancellation recorded | FEAT-23.SPEC-006 (Cancel Subscription) | Fires after status is set to Cancelled -- ends at period end | Plan reference, new status |

## Processing Logic

1. Receive the freelancer's plan reference and the trigger that fired. For the two plan-change triggers, read the live active_client_count from FEAT-01; for the count trigger, use the count supplied.
2. Read the plan's current tier and status.
3. If tier is not Paid, or status is not Active (that is, Charge failed, Cancelled -- ends at period end, or Lapsed, or the plan is Free), the plan is ineligible: if the downgrade-eligible flag is currently raised, clear it; then stop. Downgrade eligibility applies only to a currently Paid, Active plan -- a plan in a failed-charge, ending, or lapsed state is never offered a downgrade.
4. Compare active_client_count against platform parameter: `free-tier-active-client-limit`.
5. If active_client_count is at or below the threshold, raise the downgrade-eligible flag on the plan (if not already raised). When the flag goes from cleared to raised, report the offer as a record-worthy event (event type, plan reference, active_client_count, timestamp, actor "Automatic") to FEAT-13.SPEC-003 (Activity Entry Recording); this report is not repeated while the flag stays raised, and a FEAT-13 retry never delays the offer.
6. If active_client_count is above the threshold and the flag is currently raised, clear it -- the client count rose back above the threshold before Nadia acted.
7. Make the current flag state available to FEAT-23.SPEC-001 for display on next view.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Downgrade offer raised | Paid, Active plan, active_client_count at or below the free-tier threshold, flag not yet raised | Downgrade-eligible flag set on the Subscription Plan; trail entry reported | Plan & Billing Screen shows the optional downgrade offer on next view | FEAT-23.SPEC-001, FEAT-13.SPEC-003 |
| Downgrade offer cleared (count) | Active_client_count rises back above the threshold while the flag is set | Downgrade-eligible flag cleared | Plan & Billing Screen no longer shows the offer on next view | FEAT-23.SPEC-001 |
| Downgrade offer cleared (plan no longer eligible) | The plan's tier or status changes so it is not Paid and Active (Charge failed, Cancelled -- ends at period end, Lapsed, or Free after an accepted downgrade or period end) while the flag is set | Downgrade-eligible flag cleared | Plan & Billing Screen no longer shows the offer on next view | FEAT-23.SPEC-001 |
| No change | The flag's state already matches the evaluated eligibility | None | None | -- |
| Automation failure | The eligibility check itself cannot complete | Flag remains at its last known state | No user-facing error -- the offer, if any, simply continues showing its last evaluated state until the next successful check; FEAT-23.SPEC-001 additionally refuses to show or act on an offer when the plan it just read is not Paid and Active | FEAT-23.SPEC-001 |

## Data Model

**Reads:** Subscription Plan -- tier, status, active_client_count; Client (FEAT-01) -- read only to source the active-client count change event, no Client fields are written.
**Creates:** None.
**Updates:** Subscription Plan -- downgrade-eligible flag only.
**Deletes:** None.

## Business Rules

- The downgrade offer is never forced -- this automation only flags eligibility; the tier change happens only if and when Nadia accepts it through FEAT-23.SPEC-001, which routes to FEAT-23.SPEC-003 (product-features.md, Primary Flows: "Reduced usage").
- One rule for eligibility, applied identically in FEAT-23.SPEC-001, FEAT-23.SPEC-005, and FEAT-23.SPEC-007: the offer exists only while tier is Paid, status is Active, and active_client_count is at or below the free-tier threshold.
- Eligibility is re-evaluated on every active-client-count change and on every tier or status change, not on a schedule -- the offer can appear or disappear within the same session as clients are added, archived, or reactivated, or as the plan's standing changes.
- A cleared offer is not remembered as "previously offered and declined" -- if the plan is eligible again later (for example, the count drops below the threshold again, or a successful retry returns status to Active), the offer is raised again fresh (product-features.md, Primary Flows: "the offer may resurface on a later view").
- Downgrade eligibility never applies to a plan that is already Free, Charge failed, Cancelled -- ends at period end, or Lapsed -- only to a currently Paid, Active plan.

## Edge Cases

- **Active-client count drops to exactly platform parameter: `free-tier-active-client-limit`** -- Eligible; the threshold is inclusive, matching the free-tier cap's own inclusive boundary (FEAT-23.SPEC-007).
- **Nadia archives a client, becomes eligible, then reactivates a different client before viewing the offer** -- If the net count after both changes is back above the threshold, the flag is cleared before Nadia ever sees the offer; no stale offer is shown.
- **Nadia accepts the downgrade offer while a second client-count change is mid-flight (e.g., a client archive completing at the same moment)** -- The accepted downgrade proceeds through FEAT-23.SPEC-003 against the plan state as it stood when she accepted (reject-with-refresh on the plan, per the dependency map's Contention note); once the change lands as tier Free, FEAT-23.SPEC-004's tier-change trigger re-runs this automation, which clears the flag.
- **A renewal charge fails while the offer is showing** -- The status change to Charge failed fires this automation, which clears the flag; the offer disappears from the screen on next view and is raised again only after a successful retry returns status to Active with the count still at or below the threshold.
- **Concurrent trigger firing (two client-count changes, or a count change and a status change, for the same freelancer at effectively the same time)** -- Each triggers its own evaluation independently; the evaluation reading the later-committed count and plan state is authoritative, since eligibility is a re-derivable flag rather than an accumulated value.
- **Trigger fires while a previous run is in flight for the same plan** -- The second evaluation waits for the first to complete and then re-evaluates against the plan's current state at that moment, since there is exactly one flag per plan and only the latest evaluation matters; evaluations for different freelancers' plans proceed independently.
- **Automation fails to complete an evaluation** -- The flag holds its last known state (no false offer, no falsely cleared offer); the automation retries on the next trigger, and FEAT-23.SPEC-001 shows the offer's last known state with no error surfaced to Nadia, while never offering it on a plan that is not Paid and Active.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01 (Client & Project Management) | Triggered by (inbound) | Any active-client-count change (add, archive, reactivate) fires this automation |
| FEAT-23.SPEC-004 (Plan State Sync) | Triggered by (inbound) | Every committed tier or status change fires this automation |
| FEAT-23.SPEC-006 (Cancel Subscription) | Triggered by (inbound) | A recorded cancellation fires this automation so the offer is cleared |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Affects (outbound) | Displays the downgrade offer when the flag is raised |
| FEAT-23.SPEC-003 (Subscription Billing Processing) | References (outbound) | Accepting the offer routes the stop-billing request through this spec |
| FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules) | References (inbound) | Supplies the free-tier threshold and the Active-only eligibility rule this automation evaluates against |
| FEAT-13.SPEC-003 (Activity Entry Recording, FEAT-13) | Triggers (outbound) | A raised downgrade offer is reported as an append-only trail entry (Brief Cross-Feature Touchpoint); FEAT-13.SPEC-003's own trigger list is owned by FEAT-13 |

## Analytics and Success Signals

- **plan_downgrade_offered** (active_client_count, plan_tier) -- supports success-metrics.md: "Free-to-Paid Conversion" (a downgrade offer marks a freelancer moving away from paid usage, the inverse signal the conversion metric tracks against)

## Acceptance Criteria

**FEAT-23.SPEC-005-AC-01:** Given Nadia is on a Paid, Active plan with active-client count above the free-tier threshold, when she archives a client bringing her count to exactly platform parameter: `free-tier-active-client-limit`, then the downgrade-eligible flag is raised.

**FEAT-23.SPEC-005-AC-02:** Given Nadia's plan has the downgrade-eligible flag raised, when she reactivates a client bringing her count back above the threshold, then the flag is cleared.

**FEAT-23.SPEC-005-AC-03:** Given Nadia is on the Free tier, when her active-client count changes, then no downgrade-eligible flag is ever raised.

**FEAT-23.SPEC-005-AC-04:** Given Nadia's plan status is Cancelled -- ends at period end, when her active-client count drops below the threshold, then no downgrade-eligible flag is raised -- a plan already ending is not offered a further downgrade.

**FEAT-23.SPEC-005-AC-05:** Given Nadia's downgrade offer was cleared once already, when her plan is next eligible again (count at or below the threshold on a Paid, Active plan), then the offer is raised again fresh.

**FEAT-23.SPEC-005-AC-06:** Given Nadia views her Plan & Billing Screen while the downgrade-eligible flag is raised, when the screen loads, then the optional downgrade offer is shown with no forced plan change.

**FEAT-23.SPEC-005-AC-07:** Given Nadia archives one client and reactivates another in quick succession such that her net count stays above the threshold, when both changes settle, then no downgrade offer is ever shown.

**FEAT-23.SPEC-005-AC-08:** Given two of Nadia's client-count changes fire this automation at effectively the same time, when both evaluations complete, then the flag reflects the most recently committed count.

**FEAT-23.SPEC-005-AC-09:** Given this automation fails to complete an evaluation, when Nadia next views her plan, then she sees the offer's last known state with no error message, and the automation retries on the next trigger.

**FEAT-23.SPEC-005-AC-10:** Given Nadia's status is Charge failed (within the grace window) and her active-client count drops below the threshold, when this automation evaluates eligibility, then no downgrade-eligible flag is raised, since the offer exists only while status is Active.

**FEAT-23.SPEC-005-AC-11:** Given Nadia's downgrade offer is raised and she cancels instead, when FEAT-23.SPEC-006 records the cancellation, then this automation's cancellation trigger fires and the flag is cleared immediately, so the offer is not shown on the next view.

**FEAT-23.SPEC-005-AC-12:** Given Nadia's downgrade offer is raised and a renewal charge fails, when FEAT-23.SPEC-004 sets status to Charge failed, then the tier/status-change trigger fires and the flag is cleared; given a later retry returns status to Active with her count still at or below the threshold, then the offer is raised again.

**FEAT-23.SPEC-005-AC-13:** Given Nadia accepts the downgrade offer and FEAT-23.SPEC-004 sets tier to Free, when the tier-change trigger fires, then the flag is cleared and never raised while she is on the Free tier.

**FEAT-23.SPEC-005-AC-14:** Given the downgrade-eligible flag goes from cleared to raised, when it is raised, then one offer-raised event is reported to FEAT-13.SPEC-003 (Activity Entry Recording) and no further report is made while the flag stays raised.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Automation Spec: Cancel Subscription

## Overview

**Name:** Cancel Subscription
**ID:** FEAT-23.SPEC-006
**Type:** Automation
**Purpose:** Cancels Nadia's paid plan by having the subscription-billing capability acknowledge that it will not renew, then recording the cancellation, with the plan remaining Paid and fully usable through the end of the current paid period.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Handling a confirmed Cancel request from Nadia's Paid plan
- Handing the request to the billing capability (through FEAT-23.SPEC-003) and recording the cancellation only after the capability acknowledges it
- Setting status to Cancelled -- ends at period end, without changing tier or removing capacity immediately
- Defining what happens when the capability cannot acknowledge, or when the acknowledged cancellation cannot be recorded
- Reporting the recorded cancellation to the Activity & Audit Trail (FEAT-13.SPEC-003)

**Non-Goals:**
- Applying the tier change once the paid period actually ends -- owned by FEAT-23.SPEC-004 (Plan State Sync), which reacts to the period-end event this automation's relayed request eventually produces.
- Transmitting the cancellation to the subscription-billing capability and its slow/down messaging -- owned by FEAT-23.SPEC-003 (Subscription Billing Processing); this automation hands the request over and acts on the acknowledgment that comes back.
- Letting Nadia reverse a cancellation mid-period -- excluded per product-features.md: the Brief defines only Subscribe (a fresh upgrade) as the path back to Paid; no "undo cancellation" capability exists in the product definition.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia confirms Cancel in the cancellation dialog | FEAT-23.SPEC-001 (Plan & Billing Screen) | Fires only while tier is Paid and status is Active or Charge failed (FEAT-23.SPEC-007, Authorization Rules) | Plan reference, current tier, current billing_cycle |
| Cancellation acknowledged by the billing capability | FEAT-23.SPEC-003 (Subscription Billing Processing) | Fires when the capability confirms it will not renew; carries the current billing period's end date | Plan reference, period end date, acknowledgment timestamp |

## Processing Logic

One sequence, capability first: the plan is never recorded as Cancelled unless the billing capability has acknowledged the cancellation, so the record can never say "cancelled" while the capability is still set to charge.

1. Receive Nadia's confirmed cancellation request from the Plan & Billing Screen.
2. Confirm the plan's current tier is Paid and status is Active or Charge failed (re-checked authoritatively, not assumed from the screen's last load). If not, reject as stale (Outcome: Cancellation rejected -- stale state).
3. Hand the cancellation request to FEAT-23.SPEC-003 to relay to the subscription-billing capability. The plan is not changed at this point. If the capability is slow or down, FEAT-23.SPEC-003's messaging applies, no acknowledgment arrives, and the plan stays exactly as it was; Nadia may tap Cancel again later. There is no background relay retry, so no plan can be left shown as cancelled while the capability keeps charging.
4. On the capability's acknowledgment, re-read the plan. If it is still Paid with status Active or Charge failed, set status to Cancelled -- ends at period end, keep tier and billing_cycle unchanged, and store the period end date from the acknowledgment. If status was Charge failed, clear the failure record (first-failure timestamp, reason, retry attempts used) since the grace window no longer applies to a cancelled plan. If the plan has meanwhile changed (for example, it lapsed and billing was already stopped), treat the acknowledgment as moot and change nothing.
5. If the record write in step 4 fails, retry the write at platform parameter: `plan-record-write-retry-interval`, up to platform parameter: `plan-record-write-retry-count` attempts. While retrying, Nadia's screen shows "We're finishing your cancellation -- no action needed." (the acknowledgment is already in hand, so billing will not renew regardless). If all attempts fail, stop retrying (Outcome: Cancellation acknowledged but not recorded).
6. After a successful record write: notify FEAT-23.SPEC-005 so the downgrade offer is cleared, make the recorded cancellation available to FEAT-23.SPEC-001 (end-of-period explanation) and FEAT-23.SPEC-008 (cancellation confirmation email), and report the cancellation as a record-worthy event (event type, Nadia as actor, prior and new status, period end date, timestamp) to FEAT-13.SPEC-003 (Activity Entry Recording). The trail write is FEAT-13's responsibility and retries there; it never reverses the cancellation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Cancellation recorded | Capability acknowledged, and the plan was still Paid and Active or Charge failed | status=Cancelled -- ends at period end; tier and billing_cycle unchanged; period end date stored; failure record cleared if present | Plan & Billing Screen shows a plain end-of-period explanation ("Your plan stays active through {period end date}. After that, it moves to the free tier or lapses depending on your client count at that time."); confirmation email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-003, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Cancellation rejected -- stale state | Plan's state changed since Nadia's screen last loaded (e.g., already Cancelled, or a lapse already occurred) | None | Plan & Billing Screen refreshes to the current state and shows a message reflecting what actually happened: "Your plan status has changed -- here's the latest." | FEAT-23.SPEC-001 |
| Capability unavailable or no acknowledgment | The capability is slow, down, or does not acknowledge | None -- status is not changed and no relay is queued | FEAT-23.SPEC-003's messages ("Still working -- this is taking longer than usual." or "Billing is temporarily unavailable. Try cancelling again in a few minutes."); the plan continues as Paid and Active (or Charge failed) | FEAT-23.SPEC-001, FEAT-23.SPEC-003 |
| Cancellation record delayed | Capability acknowledged but the record write failed and is being retried | status unchanged until the write succeeds | Plan & Billing Screen shows "We're finishing your cancellation -- no action needed." until the write completes, then the Cancelled explanation | FEAT-23.SPEC-001 |
| Cancellation acknowledged but not recorded | The record write failed on every one of platform parameter: `plan-record-write-retry-count` attempts | status unchanged (Paid, Active or Charge failed); no email fires because no change was recorded | Plan & Billing Screen shows, in the session where Nadia cancelled, "Your billing partner has received your cancellation and will not renew your plan. Your plan ends on {period end date}; this page will show it as cancelled once our records catch up." The period-end event later reported by the capability is applied authoritatively by FEAT-23.SPEC-004, which sends the paid-plan-ended or lapse email. If Nadia taps Cancel again, the request is re-relayed and the capability's repeat acknowledgment is recorded as the same cancellation (no duplicate email or trail entry) | FEAT-23.SPEC-001, FEAT-23.SPEC-003, FEAT-23.SPEC-004 |

## Data Model

**Reads:** Subscription Plan -- current tier, status, billing_cycle, failure record.
**Creates:** None.
**Updates:** Subscription Plan -- status only (Cancelled -- ends at period end), the stored period end date, and clearing the failure record when cancelling from Charge failed.
**Deletes:** None.

## Business Rules

- Cancelling never changes tier immediately -- the plan stays Paid and fully usable through the end of the current paid period (product-features.md, Key Capabilities: "Cancel -- stop the paid plan at any time, effective at the end of the paid period").
- The billing capability's acknowledgment comes first and the local record second; the plan is never shown as Cancelled unless the capability has acknowledged it, and the screen never shows Cancelled while the capability may still renew.
- No refund or proration is calculated by this automation -- the paid period Nadia already paid for runs its full course; any refund policy question is out of scope for this feature (product-features.md's Non-Goals: independent of client-facing Invoicing & Payments).
- Cancelling is available at any time during an Active or Charge failed status -- Nadia does not need to wait for a specific point in her billing cycle to cancel. Cancelling from Charge failed ends the grace window; the plan then follows the Cancelled path to its period end.
- Once cancelled, the plan cannot be re-cancelled -- a second Cancel attempt against an already-Cancelled plan is rejected as a stale-state action (Edge Cases).

## Edge Cases

- **Nadia taps Cancel twice in quick succession** -- The second tap is disabled while the first request awaits acknowledgment; if it still arrives after the first has recorded Cancelled -- ends at period end, it is rejected as a stale-state action and the screen shows the already-cancelled state rather than double-processing.
- **The plan lapses (grace window exhausted) between Nadia loading the screen and tapping Cancel** -- The cancellation attempt is rejected as stale: Cancel is not a meaningful action against a Lapsed plan, and the screen refreshes to show the Lapsed state with its own recovery path (Subscribe again).
- **The plan lapses while the cancellation request is waiting for acknowledgment** -- The acknowledgment is treated as moot in step 4 (nothing to record); the screen refreshes to the Lapsed state.
- **Concurrent trigger firing (Nadia taps Cancel in two open sessions/tabs at the same time)** -- The first request to commit records the cancellation; the second is rejected as stale-state and the second session's screen refreshes to show the already-cancelled state.
- **Trigger fires while a previous cancellation is still awaiting the capability's acknowledgment** -- A second Cancel tap is disabled while the first is in flight (the screen's Cancel control shows a brief processing state), so no duplicate cancellation request is ever relayed for the same plan; a repeat acknowledgment that does arrive for an already-recorded cancellation is ignored.
- **Nadia's downgrade offer is showing when she cancels instead** -- The cancellation proceeds independently; once status is Cancelled -- ends at period end, this automation notifies FEAT-23.SPEC-005, whose cancellation trigger clears the flag immediately.
- **Account deletion (FEAT-24.SPEC-004) completes while the cancellation is awaiting acknowledgment or a record write is retrying** -- The Subscription Plan no longer exists; the acknowledgment and any pending retry are discarded with no effect and no email or trail entry is produced.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Triggered by (inbound) | Confirmed Cancel action initiates this automation |
| FEAT-23.SPEC-001 | Affects (outbound) | End-of-period explanation, "finishing your cancellation" notice, and stale-state refresh surface here |
| FEAT-23.SPEC-003 (Subscription Billing Processing) | Triggers (outbound) and Triggered by (inbound) | The cancellation request is relayed to the subscription-billing capability through this spec, and the capability's acknowledgment returns through it |
| FEAT-23.SPEC-004 (Plan State Sync) | References (outbound) | The eventual period-end event this cancellation produces is applied by this automation, including when the cancellation was acknowledged but not recorded |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | Triggers (outbound) | A recorded cancellation clears any raised downgrade offer |
| FEAT-23.SPEC-008 (Plan & Billing Notifications) | Triggers (outbound) | The recorded cancellation fires the cancellation confirmation email |
| FEAT-13.SPEC-003 (Activity Entry Recording, FEAT-13) | Triggers (outbound) | The recorded cancellation is reported as an append-only trail entry (Brief Cross-Feature Touchpoint); FEAT-13.SPEC-003's own trigger list is owned by FEAT-13 |
| FEAT-24 (Data Export & Account Deletion) | References (inbound) | Account deletion removes the plan; pending acknowledgments and record retries for a deleted plan are discarded |

## Analytics and Success Signals

- **subscription_cancelled** (billing_cycle, tenure_days) -- supports success-metrics.md: "Free-to-Paid Conversion" (a cancellation is the negative outcome the conversion metric's ongoing measurement tracks against)
- **cancellation_record_delayed** (attempts_used, terminal: yes / no) -- N/A -- no Stage 2 metric covers cancellation record reliability; retained so the acknowledged-but-not-recorded path is observable.

## Acceptance Criteria

**FEAT-23.SPEC-006-AC-01:** Given Nadia's plan is Paid and Active, when she confirms Cancel and the capability acknowledges, then status is set to Cancelled -- ends at period end, and tier remains Paid.

**FEAT-23.SPEC-006-AC-02:** Given Nadia's plan is now Cancelled -- ends at period end, when she views her plan, then she sees a plain explanation that it stays active through the current period's end.

**FEAT-23.SPEC-006-AC-03:** Given Nadia confirms Cancel, when the request is handed to FEAT-23.SPEC-003, then the plan's status is not changed until the capability's acknowledgment arrives.

**FEAT-23.SPEC-006-AC-04:** Given Nadia's plan is Charge failed within the grace window, when she confirms Cancel and the capability acknowledges, then the cancellation is recorded, the failure record is cleared, and the grace window no longer applies.

**FEAT-23.SPEC-006-AC-05:** Given Nadia's plan is already Cancelled -- ends at period end, when she attempts to cancel again, then the attempt is rejected as stale and the screen shows the already-cancelled state.

**FEAT-23.SPEC-006-AC-06:** Given Nadia's plan lapsed between her screen loading and her tapping Cancel, when the cancellation is attempted, then it is rejected as stale, and her screen refreshes to show the Lapsed state.

**FEAT-23.SPEC-006-AC-07:** Given Nadia taps Cancel in two open sessions at the same time, when both requests reach the automation, then only the first to commit records the cancellation and the second sees the already-cancelled state.

**FEAT-23.SPEC-006-AC-08:** Given a cancellation request is awaiting the capability's acknowledgment, when Nadia taps Cancel again, then the second tap is disabled while the first is in flight.

**FEAT-23.SPEC-006-AC-09:** Given Nadia's downgrade offer is showing when she cancels instead, when the cancellation is recorded, then FEAT-23.SPEC-005 is notified and the offer is cleared immediately, not shown on the next view.

**FEAT-23.SPEC-006-AC-10:** Given the billing capability is down when Nadia confirms Cancel, when no acknowledgment arrives, then the plan is not marked Cancelled, no background relay is queued, and she sees "Billing is temporarily unavailable. Try cancelling again in a few minutes."

**FEAT-23.SPEC-006-AC-11:** Given the capability acknowledged the cancellation but the record write fails, when the retries at platform parameter: `plan-record-write-retry-interval` are under way, then Nadia's screen shows "We're finishing your cancellation -- no action needed." and clears it once the write succeeds.

**FEAT-23.SPEC-006-AC-12:** Given the record write fails on all platform parameter: `plan-record-write-retry-count` attempts, when the last attempt fails, then retries stop, the plan status is unchanged, no email fires, Nadia's session shows the "billing partner has received your cancellation" notice with the period end date, and the capability's later period-end event is applied by FEAT-23.SPEC-004.

**FEAT-23.SPEC-006-AC-13:** Given a cancellation was acknowledged but never recorded and Nadia taps Cancel again, when the capability's repeat acknowledgment arrives and the record write succeeds, then the plan shows Cancelled once, with one confirmation email and one trail entry.

**FEAT-23.SPEC-006-AC-14:** Given a cancellation is recorded, when the write commits, then it is reported to FEAT-13.SPEC-003 as an append-only trail event with Nadia as actor, and a trail-write retry never reverses the cancellation.

**FEAT-23.SPEC-006-AC-15:** Given Nadia's account deletion completes while a cancellation acknowledgment or record retry is pending, when the pending work resolves, then it is discarded with no email and no trail entry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Plan Limit & Access Authorization Rules

## Overview

**Name:** Plan Limit & Access Authorization Rules
**ID:** FEAT-23.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs the free-tier client cap that gates client capacity, who may view versus change the Subscription Plan, and the grace/retry handling before a failed charge lapses the plan.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management
**Governed Entity:** Subscription Plan

## Scope and Non-Goals

**In Scope:**
- Field-level rules for every Subscription Plan field (tier, billing_cycle, active_client_count, status)
- The free-tier client cap threshold and its cross-field interaction with tier
- Authorization for every action on the Subscription Plan (view, subscribe, accept/decline downgrade, cancel, retry a failed charge) for every role in the Access Matrix
- The grace/retry window rule that governs when a Charge failed plan lapses
- Default values and derivations for every field

**Non-Goals:**
- The screen mechanics of viewing or acting on the plan -- owned by FEAT-23.SPEC-001 (Plan & Billing Screen), which references this spec for validation and authorization but owns its own layout and feedback.
- The mechanics of submitting a charge to the subscription-billing capability -- owned by FEAT-23.SPEC-003 (Subscription Billing Processing); this spec defines only the threshold values and access rules that spec's outcomes must respect.
- Client-side authorization for adding or reactivating a client -- owned by FEAT-01.SPEC-008 (Active Client Limit Enforcement), which reads the free-tier threshold and plan status this spec defines (XBR-23) rather than duplicating them.
- Dana (Support Operator) editing, upgrading, downgrading, or cancelling a plan -- excluded per scope-boundaries.md (SC-04) and the Access Matrix (user-persona.md): every support session is read-only, with no exception for this entity.

## Governed Entity

**Entity:** Subscription Plan
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| tier | enum (Free, Paid) | The freelancer's current subscription tier |
| billing_cycle | enum (monthly, yearly) | The paid tier's billing period; set only while tier is Paid, and unset (cleared) whenever tier is Free -- including after a downgrade, a cancellation reaching period end, or a lapse |
| active_client_count | derived (number) | The freelancer's current count of Active clients, read live from Client & Project Management (FEAT-01) |
| status | enum (Active, Charge failed, Cancelled -- ends at period end, Lapsed) | The plan's current standing |

**Valid tier and status combinations (the only five states a Subscription Plan can be in):**

| tier | status | billing_cycle | Meaning |
|------|--------|---------------|---------|
| Free | Active | unset | The starting state, or the state after a downgrade or a cancelled plan's period end with the client count within the free-tier limit |
| Paid | Active | monthly or yearly | A confirmed, billing plan |
| Paid | Charge failed | monthly or yearly (retained) | A renewal charge failed; the plan is still Paid and fully usable during the grace window. Charge failed exists only on a Paid plan -- a failed subscribe attempt never produces it (see Business Rules) |
| Paid | Cancelled -- ends at period end | monthly or yearly (retained) | Cancellation acknowledged by the billing capability; Paid and fully usable through the period end |
| Free | Lapsed | unset | The paid plan ended without being renewed (grace window exhausted, or period end reached with the active-client count over the free-tier limit). Behaves as Free for capacity: existing clients above the limit stay reachable, growth beyond the limit is blocked |

**Supporting attributes** (not primary fields of the entity; recorded and read by the specs named): last failure reason, first-failure timestamp, and retry attempts used (written by FEAT-23.SPEC-004 while status is Charge failed, cleared when it ends); downgrade-eligible flag (written by FEAT-23.SPEC-005); current billing-period end date (reported by the billing capability through FEAT-23.SPEC-003).

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-23.SPEC-001 | Plan & Billing Screen | On screen entry (visibility of actions) and on each action attempt (Subscribe, accept/decline downgrade, Cancel, Retry) |
| FEAT-23.SPEC-002 | Free Plan Auto-Provisioning | On record creation -- applies the tier, status, and billing_cycle defaults |
| FEAT-23.SPEC-004 | Plan State Sync | On every outcome it applies (charge confirmed, charge failed, grace window exhausted, period end) -- applies the status transitions and grace-window rule |
| FEAT-23.SPEC-005 | Downgrade Eligibility Detection | On every active-client-count change -- reads the free-tier threshold to evaluate eligibility |
| FEAT-01.SPEC-008 | Active Client Limit Enforcement (FEAT-01) | On client create/reactivate -- reads the free-tier threshold and current plan status defined here (XBR-23) |
| FEAT-31.SPEC-002 | Operator Support Session Console (FEAT-31) | On session open -- applies Dana's view-only authorization for plan status |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tier | Must be one of Free, Paid; never set directly by any user action -- only by FEAT-23.SPEC-002 (creation) and FEAT-23.SPEC-004 (outcome application) | Always | On write by an automation | N/A -- no user-facing input exists for this field | Yes |
| billing_cycle | Must be one of monthly, yearly when tier is Paid; must be unset when tier is Free (every write of tier=Free clears it in the same write) | Conditional on tier | On write by FEAT-23.SPEC-004, sourced from Nadia's selection captured by FEAT-23.SPEC-001 and submitted through FEAT-23.SPEC-003 | "Choose a monthly or yearly billing cycle to continue." (shown by FEAT-23.SPEC-001 if Nadia attempts to subscribe without selecting one) | Yes |
| active_client_count | No validation beyond data type -- read live from Client & Project Management (FEAT-01), never written by this feature | Always | -- | -- | -- |
| status | Must be one of Active, Charge failed, Cancelled -- ends at period end, Lapsed; never set directly by any user action -- only by FEAT-23.SPEC-004 in response to a subscription-billing outcome or the grace-window rule below | Always | On write by FEAT-23.SPEC-004 | N/A -- no user-facing input exists for this field | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Free-tier client cap | tier, active_client_count | While tier is Free, active_client_count may reach but not exceed platform parameter: `free-tier-active-client-limit`; a new or reactivated client that would exceed it is blocked at the point of creation/reactivation (enforced by FEAT-01.SPEC-008, XBR-23), not by this feature writing the count | "You've reached your plan's active client limit. Upgrade to add more clients." (shown by FEAT-01.SPEC-008's calling screen) |
| Billing cycle required on Paid | tier, billing_cycle | Whenever tier is Paid, billing_cycle must hold a value; the value is cleared in the same write that sets tier to Free, for every path that does so: downgrade, cancellation reaching period end, and lapse (grace window exhausted or period end over the limit) | "Choose a monthly or yearly billing cycle to continue." |
| Lapsed is always Free | status, tier, billing_cycle | Status Lapsed is only ever written together with tier=Free and billing_cycle cleared, on both lapse paths (grace window exhausted; period end with the count over the limit) | N/A -- applied by FEAT-23.SPEC-004, no user input |
| Charge failed only on Paid | status, tier | Status Charge failed may be set only while tier is Paid (a failed renewal charge); a failed subscribe attempt from Free or Lapsed, or a rejected downgrade request, never changes status | N/A -- applied by FEAT-23.SPEC-004, no user input |
| Lapsed blocks capacity growth | status, active_client_count | While status is Lapsed, active_client_count may not increase beyond platform parameter: `free-tier-active-client-limit` through any new or reactivated client (XBR-23); existing clients above the limit at the moment of lapse remain untouched and fully reachable | "Your plan has lapsed. Upgrade to add or reactivate clients beyond your free-tier limit." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View plan (tier, usage, billing cycle, status) | Nadia (Freelancer) | Always, her own plan only | -- |
| View plan status (status only, no billing cycle detail beyond what FEAT-31 displays) | Dana (Support Operator) | Only inside an open Support Access Session (FEAT-31.SPEC-002) | Outside a session, the plan is not reachable at all -- there is no navigation into a freelancer's account for Dana except through an open session |
| View plan | Owen (Client Primary Contact) | Never | No settings navigation from the client portal reaches this entity; the client portal product surface contains no Subscription & Account Data area (Access Matrix: None) |
| View plan | Priya (Client Reviewer Contact) | Never | Same as Owen -- no client-portal surface exists for this entity |
| Subscribe (upgrade to Paid) | Nadia (Freelancer) | Only while tier is Free -- this includes status Lapsed, which is always tier Free, so the Upgrade action offered on a Lapsed plan is authorized; never while tier is Paid (any status) | -- |
| Subscribe (upgrade to Paid) | Dana (Support Operator) | Never | No Subscribe control is ever rendered in the read-only support session (FEAT-31.SPEC-003); a direct attempt is blocked by the same session-wide read-only enforcement applied to every mutating control |
| Accept or decline the downgrade offer | Nadia (Freelancer) | Only while tier is Paid, status is Active, and the offer is currently raised (FEAT-23.SPEC-005). The offer is never raised, and is cleared, while status is Charge failed, Cancelled -- ends at period end, or Lapsed | -- |
| Accept or decline the downgrade offer | Dana (Support Operator) | Never | Same read-only enforcement as Subscribe |
| Cancel subscription | Nadia (Freelancer) | Only while tier is Paid and status is Active or Charge failed | -- |
| Cancel subscription | Dana (Support Operator) | Never | Same read-only enforcement as Subscribe |
| Retry a failed charge | Nadia (Freelancer) | Only while status is Charge failed, the grace window (platform parameter: `subscription-charge-grace-window-days`) has not elapsed, and retry attempts used are fewer than platform parameter: `subscription-charge-retry-count` | When retries are exhausted the Retry control is not rendered and the banner reads "You've used all your retries. Your plan stays fully usable until {grace_window_end_date}, then it lapses. You can cancel your plan at any time." After the window elapses the plan is already Lapsed and no Retry exists |
| Retry a failed charge | Dana (Support Operator) | Never | Same read-only enforcement as Subscribe |
| Add or reactivate an active client beyond platform parameter: `free-tier-active-client-limit` | Nadia (Freelancer) | Only while tier is Paid and status is Active, Charge failed (within the grace window), or Cancelled -- ends at period end (through the stored period end date, while the plan is still Paid and fully usable) | Save blocked on FEAT-01's add-client/reactivate screen with "You've reached your plan's active client limit. Upgrade to add more clients." (FEAT-01.SPEC-008) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| tier | Free | On record creation (FEAT-23.SPEC-002) | No -- changes only through Subscribe (to Paid), or downgrade, cancellation reaching period end, or lapse (each to Free), applied by FEAT-23.SPEC-004 |
| billing_cycle | Unset | On record creation, and again on every write that sets tier to Free | No -- set only when Nadia selects a cycle while subscribing |
| status | Active | On record creation | No -- changes only through subscription-billing outcomes (FEAT-23.SPEC-004) or the grace-window rule |
| active_client_count | Live count of Active clients from Client & Project Management (FEAT-01) | Always -- re-derived at every read, never cached across a plan or client-roster change | No -- this feature never writes this field |

## Business Rules

- XBR-23: Adding or reactivating an active client beyond the free-tier limit requires an active paid plan; when a paid plan ends, no data is lost and existing portals stay reachable, but adding clients beyond the limit is blocked. This spec owns the free-tier threshold and the plan-status conditions FEAT-01.SPEC-008 reads to enforce XBR-23.
- The free-tier limit is a platform-set policy value and is referenced only as platform parameter: `free-tier-active-client-limit`; Pass D's reconciler collects this marker into `specifications/platform-parameters.md` with a proposed default, and Gate A reconciles it against that registry. FEAT-01.SPEC-008 already references this exact marker -- this spec is its authoritative source, not a second definition of the same value.
- Grace/retry window: while status is Charge failed, the plan remains fully usable -- no mid-session lockout of already-active client work (Shared UI Pattern, Feature Breakdown Brief). Nadia may retry the charge up to platform parameter: `subscription-charge-retry-count` times within platform parameter: `subscription-charge-grace-window-days` of the first failure; FEAT-23.SPEC-004 counts each retry attempt (successful or failed) against that limit. Once the count is used up, the Retry control disappears and the banner explains it (Authorization Rules), but exhausting the count does not shorten the window: the plan stays Paid, Charge failed, and fully usable until the window ends, then FEAT-23.SPEC-004 lapses it (tier=Free, status=Lapsed, billing_cycle cleared). If no retry succeeds within the window, the same lapse applies.
- Failed-charge scope: the grace/retry window, the Charge failed status, and the failed-charge alert apply only to a failed renewal charge on a Paid plan. A failed subscribe attempt (from Free or Lapsed) leaves tier, status, and billing_cycle exactly as they were, starts no grace window, and cannot lapse anything; Nadia sees the failure reason inline and may try again without limit. A downgrade is not a charge -- if the billing capability rejects the stop-billing request, the plan is unchanged and Nadia may try again.
- Downgrade result: accepting the offer takes the plan from Paid to Free immediately once the billing capability confirms billing has stopped (tier=Free, status=Active or Lapsed per the count at that moment, billing_cycle cleared). There is no downgrade charge, and the unused remainder of the current paid period is not prorated or refunded.
- A plan that reaches Lapsed never loses existing data and never removes reachability of already-active client portals; it only blocks growth of the active-client count beyond the free-tier limit until Nadia upgrades again or archives clients (per the Brief's Primary Flows).
- Dana's read-only exception applies uniformly across this entity's every field and action -- there is no field or action for which Dana gains write access, consistent with FEAT-31.SPEC-003's session-wide read-only enforcement.
- Client-facing roles (Owen, Priya) have no authorization row that is ever exercised, because no client-portal navigation reaches this entity at all -- this reflects scope-boundaries.md (SC-01): the product models solo freelancer accounts with no billing surface for client contacts.

## Edge Cases

- **active_client_count sits exactly at platform parameter: `free-tier-active-client-limit` on the Free tier** -- Allowed; the cap is inclusive of the limit itself. Adding one more client beyond it is what triggers the block.
- **Nadia's charge fails on the very last day of platform parameter: `subscription-charge-grace-window-days`** -- A retry attempted before the window's end moment is accepted; an attempt strictly after the window's end moment is rejected and the plan transitions to Lapsed instead (FEAT-23.SPEC-004 applies the boundary as inclusive of the window's end).
- **Nadia's active-client count drops below the free-tier limit while status is Charge failed (mid-grace-window)** -- The grace-window rule and the downgrade-eligibility rule (FEAT-23.SPEC-005) evaluate independently: the charge-failed retry path continues unaffected, and because the offer is only ever raised while status is Active, the offer is raised (by FEAT-23.SPEC-005's status-change trigger) only once a successful retry returns status to Active; the count already sits at or below the limit at that moment.
- **Dana opens a support session on Nadia's account while Nadia is mid-Subscribe on her own screen** -- Dana's view is a read-only mirror; her session shows the plan's state as of its own load and never intercepts or blocks Nadia's own in-progress action, consistent with the dependency map's Contention note that billing-capability status reports (not a concurrent viewer) are authoritative for charge outcomes.
- **A new client is added at the exact moment tier transitions from Paid to Free at period end** -- The client-limit check (FEAT-01.SPEC-008) re-evaluates against the plan's current state at the moment of commit (reject-with-refresh, per the dependency map's Contention note for Subscription Plan); if the transition to Free already completed and the resulting count would exceed platform parameter: `free-tier-active-client-limit`, the add is blocked with the standard limit message.
- **Owen or Priya attempts to reach a Subscription & Account Data URL directly (out-of-scope navigation)** -- There is no such surface in the client portal product; per the Access Matrix's stated unauthorized experience, this is treated the same as any out-of-scope link: a plain explanation and no data of any kind is shown.

## Acceptance Criteria

**FEAT-23.SPEC-007-AC-01:** Given Nadia is on the free tier with active_client_count exactly at platform parameter: `free-tier-active-client-limit`, when she views her plan, then the usage display shows her at the limit and no error is present.

**FEAT-23.SPEC-007-AC-02:** Given Nadia is on the free tier at platform parameter: `free-tier-active-client-limit`, when she attempts to add one more active client (FEAT-01.SPEC-008), then the save is blocked with "You've reached your plan's active client limit. Upgrade to add more clients."

**FEAT-23.SPEC-007-AC-03:** Given Nadia's plan is Paid, when the plan record is read, then billing_cycle holds a value (monthly or yearly); given her plan is Free, then billing_cycle is unset.

**FEAT-23.SPEC-007-AC-04:** Given Nadia's status is Lapsed, when she attempts to reactivate an archived client that would push active_client_count beyond platform parameter: `free-tier-active-client-limit`, then the attempt is blocked with "Your plan has lapsed. Upgrade to add or reactivate clients beyond your free-tier limit."

**FEAT-23.SPEC-007-AC-05:** Given Nadia is viewing her own plan, when the screen loads, then she sees full tier, usage, billing cycle, and status detail.

**FEAT-23.SPEC-007-AC-06:** Given Dana has opened a support session on Nadia's account (FEAT-31.SPEC-002), when she views the Subscription & Account Data area, then she sees plan status only, with no Subscribe, downgrade, Cancel, or Retry control rendered.

**FEAT-23.SPEC-007-AC-07:** Given Dana has no open support session on any account, when she attempts to reach a freelancer's Subscription Plan, then no such navigation exists -- the plan is unreachable outside a session.

**FEAT-23.SPEC-007-AC-08:** Given Owen is signed in to his client portal, when he looks for any Subscription & Account Data navigation, then none is shown -- the client portal surface contains no such area.

**FEAT-23.SPEC-007-AC-09:** Given Priya is signed in to her client portal, when she looks for any Subscription & Account Data navigation, then none is shown, identically to Owen.

**FEAT-23.SPEC-007-AC-10:** Given Nadia's tier is Free, when she taps Subscribe and selects a billing cycle, then the request proceeds; given she is already Paid, then no Subscribe control is shown.

**FEAT-23.SPEC-007-AC-11:** Given Nadia's tier is Paid, status is Active, and a downgrade offer has been raised (FEAT-23.SPEC-005), when she views her plan, then Accept and Decline controls are both available; given no offer is currently raised, or status is Charge failed, Cancelled -- ends at period end, or Lapsed, then neither control is shown.

**FEAT-23.SPEC-007-AC-12:** Given Nadia's tier is Paid and status is Active, when she taps Cancel, then the cancellation proceeds (FEAT-23.SPEC-006); given her tier is already Free, then no Cancel control is shown.

**FEAT-23.SPEC-007-AC-13:** Given Nadia's status is Charge failed within platform parameter: `subscription-charge-grace-window-days` of the first failure and with retry attempts used below platform parameter: `subscription-charge-retry-count`, when she taps Retry, then the retry is submitted; given the grace window has elapsed, then no Retry control is shown and the plan has already transitioned to Lapsed.

**FEAT-23.SPEC-007-AC-14:** Given a charge fails at the exact instant that would be platform parameter: `subscription-charge-grace-window-days` after the first failure, when Nadia attempts a retry at that exact instant, then the retry is accepted (the boundary is inclusive).

**FEAT-23.SPEC-007-AC-15:** Given a charge fails and no retry succeeds within platform parameter: `subscription-charge-grace-window-days`, when the window closes, then the plan transitions to Lapsed and existing client work stays fully accessible.

**FEAT-23.SPEC-007-AC-16:** Given Nadia's status is Charge failed, when she continues working with an already-active client during the grace window, then nothing about her client work is blocked mid-session.

**FEAT-23.SPEC-007-AC-17:** Given Nadia's active-client count is exactly at platform parameter: `free-tier-active-client-limit` at the moment her plan transitions from Paid to Free at period end, when a client add is attempted immediately afterward, then it succeeds; one more beyond that is blocked.

**FEAT-23.SPEC-007-AC-18:** Given Nadia is mid-Subscribe on her own screen, when Dana opens a support session on the same account, then Dana's read-only view reflects the plan's state as of her session's own load and never interrupts Nadia's in-progress action.

**FEAT-23.SPEC-007-AC-19:** Given Dana has an open support session, when she attempts any action that would mutate the Subscription Plan through direct manipulation, then the attempt is blocked by the same session-wide read-only enforcement (FEAT-31.SPEC-003) applied to every other control.

**FEAT-23.SPEC-007-AC-20:** Given Nadia's status is Lapsed (tier Free), when she views her plan, then the Subscribe/Upgrade action is authorized and available, and a Subscribe request proceeds as it does on any Free plan.

**FEAT-23.SPEC-007-AC-21:** Given Nadia's status is Charge failed and her retry attempts used equal platform parameter: `subscription-charge-retry-count` with the grace window still open, when she views her plan, then no Retry control is shown, the banner reads "You've used all your retries. Your plan stays fully usable until {grace_window_end_date}, then it lapses. You can cancel your plan at any time.", and the plan lapses at the window's end rather than at the moment the count ran out.

**FEAT-23.SPEC-007-AC-22:** Given Nadia's plan lapses by either path, when the lapse is written, then tier is Free, status is Lapsed, and billing_cycle is unset in that same write.

**FEAT-23.SPEC-007-AC-23:** Given Nadia's plan is Free and her Subscribe attempt fails, when the failure is reported, then her status stays as it was (Active or Lapsed), no grace window starts, and she may attempt again with no retry limit.

**FEAT-23.SPEC-007-AC-24:** Given Nadia accepts a downgrade and the billing capability confirms billing has stopped, when FEAT-23.SPEC-004 applies it, then tier is Free, billing_cycle is unset, and no charge was made; given the capability rejects the request, then the plan stays Paid and Active.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 5 | 5 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |



# Notification Spec: Plan & Billing Notifications

## Overview

**Name:** Plan & Billing Notifications
**ID:** FEAT-23.SPEC-008
**Type:** Notification
**Purpose:** Sends Nadia an email confirmation on every plan change (upgrade, downgrade, cancellation, lapse) and an alert when a subscription charge fails.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- The plan-change confirmation email, sent on upgrade, downgrade (Paid to Free, effective immediately), cancellation confirmed, paid plan ended at period end (moved to Free), and lapse
- The failed-charge alert email, sent when a renewal charge on a Paid plan first fails (never for a failed subscribe attempt or a rejected downgrade, which are shown inline and change nothing)
- Delivery, retry, and expiry behavior for both, using the transactional email delivery capability (FEAT-14.SPEC-001)

**Non-Goals:**
- Routine renewal notifications -- excluded per FEAT-23.SPEC-004's Business Rules: a successful renewal is silent by design; only a tier or status change is worth interrupting Nadia for.
- The mechanics of the transactional email delivery capability itself -- owned by FEAT-14.SPEC-001 (Transactional Email Delivery), which this spec uses without duplicating its contract.
- General account notification preferences (turning all email off, quiet hours across every feature) -- owned by FEAT-21.SPEC-002 (Notification Preferences); this spec defines only what is specific to plan and billing events, both of which are transactional and always send (XBR-30, via FEAT-21.SPEC-008).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every plan-change and failed-charge event | Nadia may not be inside the product at the moment her billing state changes (a renewal failure, a period ending); a billing outcome that affects whether she can add clients must reach her even when she is away from the product (ASMP-26: failed deliveries must be surfaced within minutes, not lost) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Plan tier or status changes (upgrade, downgrade, cancellation confirmed, paid plan ended at period end, or lapse) | FEAT-23.SPEC-004 (Plan State Sync), FEAT-23.SPEC-006 (Cancel Subscription) | Fires on every tier/status change this spec's In Scope covers -- never on routine renewal, never on a charge recovered by a successful retry, and never on a failed subscribe attempt or rejected downgrade. A downgrade confirmed while the live client count is over the free-tier limit results in Lapsed and sends the lapse email instead of the downgrade email | Plan reference, prior tier/status, new tier/status, billing_cycle, period end date (cancellation and period end), failure reason (lapse only, if applicable) |
| A renewal charge first fails | FEAT-23.SPEC-003 (Subscription Billing Processing), via FEAT-23.SPEC-004 | Fires once when a Paid, Active plan's renewal charge does not complete and status becomes Charge failed; not fired for failed retries within the same grace window, and not for a subscribe or downgrade attempt | Plan reference, specific failure reason, grace-window end date |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient per the Access Matrix in user-persona.md; this is her own billing relationship, never visible to Owen, Priya, or Dana by email (Dana's status-only view is inside a support session, per FEAT-31, and carries no email of its own).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Plan-change and failed-charge emails | Always on (transactional -- cannot be turned off) | On | N/A -- FEAT-21.SPEC-002 (Notification Preferences) lists this as a locked-on transactional category rather than a toggle, per FEAT-21.SPEC-008 (Notification Preference Rules) and XBR-30 |

**Quiet Hours:** N/A -- these are transactional, account-standing emails (a billing outcome affecting whether Nadia can add clients), exempt from any quiet-hours hold, consistent with how FEAT-21.SPEC-008 treats every transactional record email.

## Content Definition

**Email (plan-change confirmation -- upgrade):**
- **Subject:** Your Clientroom plan is now {new_tier_label}
- **Body:**
  Hi {nadia_first_name},

  Your Clientroom plan has changed to {new_tier_label} ({billing_cycle_label}). You can now have up to {new_client_capacity} active clients.

  Review your plan and billing details any time.
- **CTA (button):** View plan -- deep-links to FEAT-23.SPEC-001 (Plan & Billing Screen)

**Email (plan-change confirmation -- downgrade):**
- **Subject:** Your Clientroom plan is now {new_tier_label}
- **Body:**
  Hi {nadia_first_name},

  Your Clientroom plan has changed to {new_tier_label}, effective now. Billing has stopped and you won't be charged again. Your active client limit is now {new_client_capacity}.

  Your existing clients and their portals are untouched. Review your plan and billing details any time.
- **CTA (button):** View plan -- deep-links to FEAT-23.SPEC-001

**Email (plan-change confirmation -- cancellation confirmed):**
- **Subject:** Your Clientroom plan cancellation is confirmed
- **Body:**
  Hi {nadia_first_name},

  Your subscription is cancelled. Your plan stays fully active through {period_end_date}. After that, it moves to the free tier or lapses depending on your active client count at that time.

  You can resubscribe at any point.
- **CTA (button):** View plan -- deep-links to FEAT-23.SPEC-001

**Email (plan-change confirmation -- paid plan ended at period end):**
- **Subject:** Your Clientroom paid plan has ended
- **Body:**
  Hi {nadia_first_name},

  Your paid plan ended on {plan_end_date}, as you asked when you cancelled. You're now on the Free tier, with room for up to {free_tier_client_limit} active clients. Nothing has been lost -- your existing clients and their portals are untouched.

  You can subscribe again at any point.
- **CTA (button):** View plan -- deep-links to FEAT-23.SPEC-001 (Plan & Billing Screen)

Sent when FEAT-23.SPEC-004 applies a period end with the client count within the free-tier limit (tier Free, status Active). When the count is over the limit the period end results in Lapsed and the lapse email below is sent instead; exactly one email is sent for a given period end. A cancellation therefore produces two emails over time by design: the cancellation-confirmed email when Nadia cancels, and this one (or the lapse email) on the day the plan actually ends.

**Email (plan-change confirmation -- lapsed):**
- **Subject:** Your Clientroom plan has lapsed
- **Body:**
  Hi {nadia_first_name},

  Your plan has lapsed as of {lapse_date}. Nothing has been lost -- your existing clients and their portals remain reachable. Adding or reactivating clients beyond {free_tier_client_limit} is on hold until you upgrade again.
- **CTA (button):** Upgrade -- deep-links to FEAT-23.SPEC-001 (the Subscribe action)

**Email (failed-charge alert):**
- **Subject:** We couldn't process your Clientroom subscription charge
- **Body:**
  Hi {nadia_first_name},

  Your subscription charge didn't go through: {failure_reason}. Your plan and client access are unaffected for now -- you have until {grace_window_end_date} to retry before your plan lapses.
- **CTA (button):** Retry now -- deep-links to FEAT-23.SPEC-001 (the Retry action)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|----------------|-----------------------|
| {nadia_first_name} | Freelancer Account -- first name | Priya | Greeting renders as "Hi," |
| {new_tier_label} | Subscription Plan -- tier (rendered as "Paid" or "Free") | Paid | Never empty -- tier is always set on the plan record |
| {billing_cycle_label} | Subscription Plan -- billing_cycle (rendered as "billed monthly" or "billed yearly") | billed yearly | Line is omitted when billing_cycle is unset (e.g., downgrade to Free -- not applicable there since that email variant omits the clause) |
| {new_client_capacity} | Derived -- platform parameter: `free-tier-active-client-limit` for Free, unlimited for Paid (rendered as "unlimited" on Paid) | unlimited | Never empty -- always derivable from tier |
| {period_end_date} | Subscription Plan -- current billing period's end date, reported by the subscription-billing capability (FEAT-23.SPEC-003) | March 14, 2027 | Never empty -- a cancellation is only recorded against an active billing period that has a known end date |
| {plan_end_date} | Subscription Plan -- the period end date the capability reported, as applied by FEAT-23.SPEC-004 | March 14, 2027 | Never empty -- the period-end event carries its own date |
| {lapse_date} | Derived -- the date this spec's trigger event was applied | September 27, 2026 | Never empty -- the lapse event itself carries its own timestamp |
| {free_tier_client_limit} | Derived -- platform parameter: `free-tier-active-client-limit` | 2 | Never empty -- a fixed platform value |
| {failure_reason} | Subscription Plan -- last failure reason, reported by the subscription-billing capability (FEAT-23.SPEC-003) | Your card was declined | "a billing issue" (generic fallback if no specific reason was reported) |
| {grace_window_end_date} | Derived -- first-failure timestamp plus platform parameter: `subscription-charge-grace-window-days` | October 4, 2026 | Never empty -- calculable the moment the failure is recorded |

## Delivery Rules

**Batching:** None -- each plan-change or failed-charge event is delivered as its own, individual email the moment it is recorded; these are infrequent, high-importance events that are never worth collapsing together.
**Deduplication:** At most one email per distinct tier/status-change event or per distinct charge-failure event. A charge-succeeded event already applied and reported does not re-fire its confirmation if the same event is delivered twice to FEAT-23.SPEC-004 (per that spec's own deduplication).
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning inside the product (XBR-30); the underlying plan/status change itself remains visible on the Plan & Billing Screen (FEAT-23.SPEC-001) regardless of this email's delivery outcome.
**Expiry:** These emails never expire undelivered in the ordinary sense -- because the underlying tier/status change or failure reason remains visible on the Plan & Billing Screen indefinitely, a late-delivered confirmation still carries accurate, current information; delivery is retried until it succeeds or the retry count is exhausted, never abandoned as "too late to matter."

## Edge Cases

- **Nadia's account is deleted (FEAT-24) before a pending confirmation email is delivered** -- The pending delivery is cancelled silently; an email about a plan that no longer exists is never sent.
- **Nadia's subscribe attempt fails, or her downgrade request is rejected** -- No email is sent: the outcome is shown inline on the Plan & Billing Screen (FEAT-23.SPEC-001), the plan is unchanged, and no grace window exists that an alert could point to.
- **A cancelled plan reaches its period end** -- One email is sent for that period end: the paid-plan-ended email (Free) or the lapse email (Lapsed), in addition to the cancellation-confirmed email sent earlier when Nadia cancelled.
- **A failed-charge alert is still pending delivery when the retry succeeds** -- The pending failed-charge alert is cancelled if it has not yet sent, and the plan-change confirmation for the successful retry (status returning to Active, no tier change) is not sent either, since a return to the prior status without a tier change is not itself a plan change worth confirming; only a genuine tier/status change (upgrade, downgrade, cancellation, or lapse) triggers this spec.
- **A plan lapses and Nadia immediately re-subscribes before the lapse email is delivered** -- Both emails are sent independently in their own right (the lapse happened and is worth confirming; the upgrade is a separate, later event), since each documents a real state the plan passed through -- this reflects the product's commitment to a truthful, evidentiary trail rather than suppressing history for tidiness.
- **The same failure reason repeats across multiple retry attempts within the grace window** -- Only the first failed-charge alert for a given grace window is sent; repeated retry failures within the same still-open grace window do not generate additional alerts, since Nadia already has the grace-window end date and a working Retry action from the first alert.
- **Email delivery fails on all retries for a lapse confirmation** -- The lapse itself is fully recorded and visible on the Plan & Billing Screen (FEAT-23.SPEC-001) regardless of the email's fate; a delivery warning is added to Nadia's account per XBR-30, so the change is never silently lost even if the email never arrives.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-004 (Plan State Sync) | Triggered by (inbound) | Every tier/status change (except renewal) and every charge-failed event fires this notification |
| FEAT-23.SPEC-006 (Cancel Subscription) | Triggered by (inbound) | A recorded cancellation fires the cancellation-confirmed email |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Navigation (outbound) | Every CTA deep-links here |
| FEAT-14.SPEC-001 (Transactional Email Delivery, FEAT-14) | Triggers (outbound) | This spec's emails are sent through that capability, which reports delivery and bounce status back |
| FEAT-21.SPEC-008 (Notification Preference Rules, FEAT-21) | References (inbound) | Confirms these emails are locked-on transactional emails per XBR-30, never optional |

## Analytics and Success Signals

- **plan_change_email_sent** (change_type: upgrade / downgrade / cancellation / plan_ended / lapse) -- supports success-metrics.md: "Free-to-Paid Conversion" (documents the confirmed outcomes the conversion metric tracks)
- **failed_charge_alert_sent** (failure_reason) -- N/A -- no Stage 2 metric measures alert volume directly; retained so failure-alert delivery is observable alongside FEAT-23.SPEC-003's own subscription_charge_failed event.
- **plan_change_email_delivery_failed** (change_type) -- N/A -- no Stage 2 metric covers email-delivery reliability for this feature specifically; ASMP-26's minutes-not-lost commitment is the product-level requirement this event supports observability for.

## Acceptance Criteria

**FEAT-23.SPEC-008-AC-01:** Given Nadia's plan upgrades to Paid, when the upgrade is applied (FEAT-23.SPEC-004), then she receives an email "Your Clientroom plan is now Paid" naming her new client capacity.

**FEAT-23.SPEC-008-AC-02:** Given Nadia accepts a downgrade and it is applied (tier Free, status Active), when FEAT-23.SPEC-004 commits it, then she receives "Your Clientroom plan is now Free" stating the change is effective now, billing has stopped, and naming her new client limit.

**FEAT-23.SPEC-008-AC-03:** Given Nadia cancels her subscription, when the cancellation is recorded (FEAT-23.SPEC-006), then she receives "Your Clientroom plan cancellation is confirmed" stating the exact date her plan stays active through.

**FEAT-23.SPEC-008-AC-04:** Given Nadia's plan lapses, when the lapse is applied, then she receives "Your Clientroom plan has lapsed" stating that no data is lost and existing clients remain reachable.

**FEAT-23.SPEC-008-AC-05:** Given a Paid, Active plan's renewal charge fails, when the failure is recorded, then Nadia receives "We couldn't process your Clientroom subscription charge" naming the specific reason and the grace-window end date.

**FEAT-23.SPEC-008-AC-06:** Given Nadia's plan renews successfully, when the renewal event is applied, then no confirmation email is sent for it.

**FEAT-23.SPEC-008-AC-07:** Given Nadia's account is deleted before a pending confirmation email is delivered, when the deletion completes, then the pending email is cancelled and never sent.

**FEAT-23.SPEC-008-AC-08:** Given a failed-charge alert is pending delivery, when Nadia's retry succeeds before it sends, then the pending alert is cancelled and no confirmation email is sent for the return to Active status alone.

**FEAT-23.SPEC-008-AC-09:** Given Nadia's plan lapses and she immediately re-subscribes, when both events are recorded, then she receives both the lapse email and the upgrade confirmation email, each independently.

**FEAT-23.SPEC-008-AC-10:** Given a charge fails twice within the same still-open grace window, when the second failure is recorded, then no second failed-charge alert is sent.

**FEAT-23.SPEC-008-AC-11:** Given the lapse confirmation email fails delivery on every retry, when the final retry fails, then the lapse itself remains fully visible on the Plan & Billing Screen and a delivery warning is recorded per XBR-30.

**FEAT-23.SPEC-008-AC-12:** Given these are transactional emails, when Nadia looks in her Notification Preferences (FEAT-21.SPEC-002), then plan-change and failed-charge emails show as always-on with no toggle.

**FEAT-23.SPEC-008-AC-13:** Given a plan-change email is triggered during Nadia's own quiet hours preference window (set for other notification types), when it is due to send, then it sends immediately regardless, since these emails are exempt from quiet hours.

**FEAT-23.SPEC-008-AC-14:** Given a cancelled plan reaches its period end with Nadia's active-client count within the free-tier limit, when FEAT-23.SPEC-004 applies it, then she receives "Your Clientroom paid plan has ended" stating the {plan_end_date}, that she is now on the Free tier with room for {free_tier_client_limit} active clients, and that nothing was lost.

**FEAT-23.SPEC-008-AC-15:** Given a cancelled plan reaches its period end with the count over the free-tier limit, when FEAT-23.SPEC-004 applies it, then she receives only the lapse email, not the paid-plan-ended email.

**FEAT-23.SPEC-008-AC-16:** Given Nadia cancelled earlier and her plan later reaches its period end, when both events have been applied, then she has received two emails in total for the cancellation journey: the cancellation-confirmed email at cancellation and one period-end email (paid-plan-ended or lapse) at the end.

**FEAT-23.SPEC-008-AC-17:** Given Nadia's subscribe charge fails or her downgrade request is rejected, when the outcome is reported, then no email is sent and the reason appears inline on the Plan & Billing Screen only.

**FEAT-23.SPEC-008-AC-18:** Given a failed retry occurs within an open grace window, when FEAT-23.SPEC-004 records it, then no further failed-charge alert is sent, and a successful retry sends no plan-change email.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 | 2 |
| Preference States | 1 (locked-on transactional) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
