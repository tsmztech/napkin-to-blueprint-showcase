---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-23.SPEC-007
spec_name: Plan Limit & Access Authorization Rules
spec_slug: plan-limit-access-authorization-rules
parent_feature: FEAT-23
parent_feature_name: Subscription Plan & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 24
---

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
