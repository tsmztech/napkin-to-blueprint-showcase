---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-04.SPEC-009
spec_name: Swap Concurrency Lock
spec_slug: swap-concurrency-lock
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 11
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Swap Concurrency Lock

## Overview

**Name:** Swap Concurrency Lock
**ID:** FEAT-04.SPEC-009
**Type:** Logic/Rule
**Purpose:** Enforces at most one active swap operation per meal slot at a time, preventing duplicate swaps from a double tap or a race between a direct swap and an accepted suggestion.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap
**Governed Entity:** Planned Meal (the slot's in-flight-swap state, guarding writes to its status and recipe fields)

## Scope and Non-Goals

**In Scope:**
- Acquiring and releasing a per-slot lock around any operation that would write a new recipe onto a Planned Meal
- Defining exactly what a second, concurrent attempt on the same slot experiences while the lock is held
- Authorization for who may acquire this lock

**Non-Goals:**
- Deciding which candidates are safe to swap to -- owned by FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation); this spec governs only the write-time exclusivity, not candidate eligibility
- Performing the write itself -- owned by FEAT-04.SPEC-004 (Apply Meal Swap), which acquires this lock before writing and releases it afterward
- Locking anything at the Weekly Plan level or across different slots -- excluded per product-features.md's Validation & Limits, which scopes the "no more than one active swap operation" rule to "per meal slot"; swaps on different nights never contend with each other
- Preventing a suggestion from being created while a slot is locked -- a new suggestion (FEAT-04.SPEC-010) does not itself write to Planned Meal, so submitting one is never blocked by this lock; only the act of applying a swap is

## Governed Entity

**Entity:** Planned Meal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| night | text/enum | The day of the week this slot occupies -- the scope unit this lock is keyed to |
| meal_kind | enum | Dinner, or leftover lunch linked to a source dinner |
| recipe | reference | The field this lock protects against a concurrent overwrite |
| safety_badge | derived | Not governed by this spec |
| vegetarian_option | boolean | Not governed by this spec |
| cook_time | derived | Not governed by this spec |
| rough_cost | derived | Not governed by this spec |
| pantry_callout | derived | Not governed by this spec |
| status | enum | The field this lock protects against a concurrent double-write to Swapped |
| swap_history | list | Not governed by this spec directly, though it receives the write this lock protects |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-04.SPEC-001 | Meal Swap Direct | Acquired immediately after Maya selects an alternative, before FEAT-04.SPEC-004 is invoked |
| FEAT-04.SPEC-003 | Review Swap Suggestions | Acquired immediately after Maya taps Accept, before FEAT-04.SPEC-004 is invoked |
| FEAT-04.SPEC-004 | Apply Meal Swap | Holds the lock for the duration of its processing; releases it on completion (success or failure) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status (via the lock) | At most one active swap operation may target this Planned Meal's status/recipe pair at a time | Whenever a swap-apply attempt (direct or accepted-suggestion) is made for this slot | On lock acquisition, before FEAT-04.SPEC-004 begins | "This meal is already being swapped." (shown on FEAT-04.SPEC-001) / "Couldn't complete this action. Try again." (shown on FEAT-04.SPEC-003, since the accept simply fails to proceed) | Yes |
| night, meal_kind, recipe (read-only reference), safety_badge, vegetarian_option, cook_time, rough_cost, pantry_callout, swap_history | No validation beyond data type -- these fields are not independently governed by this lock | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Lock scope is the slot, not the plan | night (as the lock key), status | The lock is keyed to one specific Planned Meal (one night, one Weekly Plan); an operation on a different night's slot in the same plan is never blocked by a lock held elsewhere in the plan | N/A -- this is a scoping rule with no error condition of its own |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Acquire the swap lock for a slot (direct swap) | Maya (Organiser) | Always, subject to the lock being free | If the lock is held: "This meal is already being swapped." shown in place of the alternatives list on FEAT-04.SPEC-001 |
| Acquire the swap lock for a slot (applying an accepted suggestion) | Maya (Organiser) | Always, subject to the lock being free | If the lock is held: the accept action on FEAT-04.SPEC-003 fails and that card shows "Couldn't complete this action. Try again." |
| Acquire the swap lock for a slot | Sam (Other Adult Member) | Never -- Sam never applies a swap directly; his path (FEAT-04.SPEC-002) creates a suggestion, which does not acquire this lock | N/A -- Sam has no path that attempts to acquire this lock |
| Acquire the swap lock for a slot | Jordan (young kid, no login), Jordan (older kid, limited login), Riley (Operator) | Never | N/A -- none of these roles has any Meal Swap access path that could attempt to acquire this lock |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Lock key | Derived as the specific Planned Meal (one Weekly Plan, one night) targeted by the swap-apply attempt | On every lock acquisition attempt | No |
| Lock hold duration | Held for the full duration of FEAT-04.SPEC-004's processing (safety re-check through write completion) and released immediately on that automation's completion, whether it succeeds or fails | Always | No |

## Business Rules

- **Scope is per meal slot, per product-features.md's Validation & Limits:** "no more than one active swap operation per meal slot at a time (prevents duplicate swaps from a double tap)." A double tap on the same alternative, and a race between a direct swap and an accepted suggestion for the same slot, are the two scenarios this rule exists to prevent.
- **The lock is released even on failure:** a failed safety re-check or a failed write (FEAT-04.SPEC-004's failure outcomes) always releases the lock so a retry can proceed; the lock is never left held after an operation ends, under any outcome.
- **Only an apply attempt acquires the lock -- a request for alternatives never does:** browsing alternatives (FEAT-04.SPEC-008) is read-only and does not contend with an in-flight swap; the lock guards only the moment a candidate is chosen and the write begins.
- **First-to-acquire wins; the second attempt is refused outright, not queued:** this rule does not make a second attempt wait for the first to finish and then proceed automatically -- the user sees the Locked message immediately and must retry once the first operation has completed, keeping the "one tap" experience honest about what actually happened rather than silently reordering swaps.

## Edge Cases

- **Maya double-taps the same alternative rapidly on FEAT-04.SPEC-001** -- The screen's own debounce (disabling other items during Applying) is the first line of defense; even if a second request reached this spec, the lock acquisition for the second would fail since the first is already held, and the second shows the Locked message.
- **A direct swap by Maya and an accepted suggestion for the same slot are triggered at effectively the same moment (from two devices)** -- Whichever acquires the lock first proceeds through FEAT-04.SPEC-004; the second is refused with the Locked message on FEAT-04.SPEC-001 or the accept-failure message on FEAT-04.SPEC-003, depending on which path lost the race. Per the dependency map's Planned Meal Contention note, this is reject-with-refresh at the slot level, and a double tap never creates two swaps.
- **The lock-holding operation fails (safety re-check or write failure)** -- The lock releases immediately per Business Rules; a subsequent attempt on the same slot is not blocked by the failed attempt.
- **A swap on one night and a swap on a different night in the same Weekly Plan are attempted simultaneously** -- Both proceed independently; the lock is scoped per slot, not per plan (Cross-Field Rules).
- **The lock-holding operation stalls indefinitely (e.g., the device loses connectivity mid-operation)** -- The lock is tied to the server-side completion of FEAT-04.SPEC-004, not to the initiating device staying connected; if that automation's processing itself times out, its own failure path (FEAT-04.SPEC-004's Write Failure outcome) releases the lock, so a stalled client never leaves the slot permanently locked.

## Acceptance Criteria

**FEAT-04.SPEC-009-AC-01:** Given no swap operation is active on a slot, when Maya selects an alternative on FEAT-04.SPEC-001, then the lock is acquired and FEAT-04.SPEC-004 proceeds.

**FEAT-04.SPEC-009-AC-02:** Given a swap operation is already active on a slot, when Maya attempts a second swap on that same slot, then the lock acquisition fails and FEAT-04.SPEC-001 shows "This meal is already being swapped."

**FEAT-04.SPEC-009-AC-03:** Given a swap operation is already active on a slot, when Maya attempts to accept a suggestion for that same slot on FEAT-04.SPEC-003, then the accept fails and that card shows "Couldn't complete this action. Try again."

**FEAT-04.SPEC-009-AC-04:** Given a direct swap and an accepted suggestion for the same slot are triggered at effectively the same time, when both attempt to acquire the lock, then exactly one proceeds and the other is refused.

**FEAT-04.SPEC-009-AC-05:** Given a lock-holding operation's safety re-check fails, when FEAT-04.SPEC-004 reports that failure, then the lock is released immediately.

**FEAT-04.SPEC-009-AC-06:** Given a lock-holding operation's write fails, when FEAT-04.SPEC-004 reports that failure, then the lock is released immediately and a retry on the same slot can proceed.

**FEAT-04.SPEC-009-AC-07:** Given swaps are attempted on two different nights in the same Weekly Plan at the same time, when both attempt to acquire their respective locks, then both proceed independently.

**FEAT-04.SPEC-009-AC-08:** Given Maya double-taps the same alternative rapidly, when the second tap registers, then it is ignored by the screen's debounce before ever reaching this spec's lock acquisition.

**FEAT-04.SPEC-009-AC-09:** Given Sam has no path that applies a swap directly, when his suggestion flow is examined, then it never attempts to acquire this lock.

**FEAT-04.SPEC-009-AC-10:** Given a lock is held while a request for alternatives (not an apply attempt) is made for the same slot, when that request is processed, then it succeeds normally -- browsing alternatives never contends with the lock.

**FEAT-04.SPEC-009-AC-11:** Given a lock-holding operation's client loses connectivity mid-operation, when the server-side processing (FEAT-04.SPEC-004) reaches its own failure path, then the lock is released rather than remaining held indefinitely.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
