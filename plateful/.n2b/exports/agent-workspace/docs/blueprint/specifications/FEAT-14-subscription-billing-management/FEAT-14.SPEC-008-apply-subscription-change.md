---
document_type: spec
spec_type: automation
spec_id: FEAT-14.SPEC-008
spec_name: Apply Subscription Change
spec_slug: apply-subscription-change
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

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
