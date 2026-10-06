---
document_type: spec
spec_type: automation
spec_id: FEAT-23.SPEC-005
spec_name: Downgrade Eligibility Detection
spec_slug: downgrade-eligibility-detection
parent_feature: FEAT-23
parent_feature_name: Subscription Plan & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

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
