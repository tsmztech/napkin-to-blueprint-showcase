---
document_type: spec
spec_type: automation
spec_id: FEAT-21.SPEC-006
spec_name: Sign-Out Other Sessions
spec_slug: sign-out-other-sessions
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Sign-Out Other Sessions

## Overview

**Name:** Sign-Out Other Sessions
**ID:** FEAT-21.SPEC-006
**Type:** Automation
**Purpose:** Invalidates every one of Nadia's signed-in sessions except the current one and records the event, when she taps "Sign out other devices".
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Invalidating every active session on Nadia's account except the session that issued the request
- Refreshing the signed-in devices list shown on FEAT-21.SPEC-003 once complete
- Recording the sign-out event

**Non-Goals:**
- Signing out any single, individually chosen device -- the Brief's Key Capabilities and FEAT-21.SPEC-003's layout define only an all-other-sessions action, not per-device selection
- Changing the sign-in email or login method -- a distinct capability owned by FEAT-21.SPEC-005; signing out other sessions never alters Nadia's sign-in email
- Signing out Dana's support session or any client contact session -- excluded because neither Dana's Support Access Session (FEAT-31) nor a Client Contact's portal session is a signed-in device on the Freelancer Account; this automation's scope is limited entirely to Nadia's own sessions

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| "Sign out other devices" confirmed | FEAT-21.SPEC-003 (Login & Security) | Fires when Nadia confirms the "Sign out other devices" action, and at least one other active session exists | Freelancer Account reference, current session identifier, list of all other active session identifiers at request time |

## Processing Logic

1. Receive the request from the triggering screen, including the identifier of the current (requesting) session.
2. Read the full list of active sessions on the Freelancer Account.
3. Exclude the current session from the list -- it is never a candidate for invalidation.
4. Invalidate every remaining session on the list immediately.
5. Record the event.
6. Signal the triggering screen that invalidation is complete, with the count of sessions signed out.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Sessions signed out | One or more other sessions existed and were invalidated | All non-current sessions on the Freelancer Account marked invalid; event recorded | FEAT-21.SPEC-003 shows toast "All other sessions signed out." and the device list refreshes to show only "This device" | FEAT-21.SPEC-003 |
| No-action (nothing to sign out) | No other active sessions existed at request time | None | Not user-visible as a distinct outcome -- FEAT-21.SPEC-003 never shows the "Sign out other devices" button when no other session exists, so this path is only reachable if the last other session ended between screen load and the request; in that case the device list simply refreshes with the toast "All other sessions signed out." (zero sessions affected) | FEAT-21.SPEC-003 |
| Automation failure | Invalidation cannot complete (e.g., the session store is unreachable) | No sessions invalidated | FEAT-21.SPEC-003 shows inline error "Could not sign out other sessions. Try again." and the device list remains unchanged | FEAT-21.SPEC-003 |

## Data Model

**Reads:** Freelancer Account -- signed-in devices list, current session identifier.
**Creates:** None -- the sign-out event is recorded, not created as a new tracked entity beyond that record.
**Updates:** Freelancer Account -- signed-in devices list (each non-current session marked invalid/removed).
**Deletes:** None -- session invalidation is a state change, not a deletion of the account record.

## Business Rules

- The current session is never included in the invalidation, by construction of the trigger's available data (the current session identifier is excluded before invalidation runs).
- This action always signs out all other sessions at once -- there is no partial or per-device variant (per the Brief's Key Capabilities: "can sign out other devices").
- A signed-out session cannot be resumed; the device must sign in again from the start, going through the product's standard sign-in path.

## Edge Cases

- **Nadia taps "Sign out other devices" with zero other sessions active (the last one ended between screen load and her tap)** -- The action still completes with zero sessions affected; the toast "All other sessions signed out." still appears since, from Nadia's perspective, the intended end state (only her current session active) is achieved.
- **A device being signed out is mid-action (e.g., another of Nadia's sessions is in the middle of saving a form) at the moment of invalidation** -- That other session's next request is rejected as unauthenticated; any unsaved data in that session is lost, consistent with an ordinary session expiry on that device.
- **Concurrent trigger firing (Nadia taps "Sign out other devices" from two of her own sessions at effectively the same time)** -- Whichever request's invalidation step commits first signs out every other session, including the second triggering session itself if it was not the one processed first; the second request then finds its own session already invalidated and the triggering screen redirects that device to sign in again.
- **Trigger fires while a previous run is in flight** -- A second "Sign out other devices" tap from the same session while the first request is still processing is ignored, since FEAT-21.SPEC-003's button shows a loading state during processing and does not accept a second tap.
- **A device reconnects immediately after being signed out** -- It is treated as a fresh, unauthenticated session and must sign in again; it is not automatically re-admitted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-003 (Login & Security) | Triggered by (inbound) | "Sign out other devices" confirmation fires this automation |
| FEAT-21.SPEC-003 (Login & Security) | Affects (outbound) | Refreshed device list and completion toast/error shown there |

## Analytics and Success Signals

- **other_sessions_signed_out** (session_count) -- N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Signals field ("other_sessions_signed_out") so this security action is observable
- **other_sessions_sign_out_failed** () -- N/A -- no connected success-metrics.md metric; retained so a failed security action is observable rather than silent

## Acceptance Criteria

**FEAT-21.SPEC-006-AC-01:** Given Nadia has two other active sessions in addition to her current one, when she confirms "Sign out other devices", then both other sessions are invalidated, her current session remains active, and she sees "All other sessions signed out."

**FEAT-21.SPEC-006-AC-02:** Given Nadia's other session was already invalidated by the time her confirmed request is processed (it ended on its own between load and request), when this automation runs, then it completes with zero sessions affected and she still sees "All other sessions signed out."

**FEAT-21.SPEC-006-AC-03:** Given a signed-out device attempts its next action after invalidation, then that action is rejected as unauthenticated and the device is returned to the sign-in screen.

**FEAT-21.SPEC-006-AC-04:** Given Nadia taps "Sign out other devices" from two of her own sessions at effectively the same time, when the first request's invalidation commits, then the second triggering session is itself signed out and is redirected to sign in again.

**FEAT-21.SPEC-006-AC-05:** Given Nadia taps "Sign out other devices" a second time while the first request is still processing, then the second tap is ignored because the button is in a loading state.

**FEAT-21.SPEC-006-AC-06:** Given the session invalidation step fails, when the failure occurs, then no sessions are invalidated and FEAT-21.SPEC-003 shows "Could not sign out other sessions. Try again."

**FEAT-21.SPEC-006-AC-07:** Given a device Nadia just signed out attempts to reconnect immediately afterward, then it is treated as a fresh, unauthenticated session and must sign in again.

**FEAT-21.SPEC-006-AC-08:** Given this automation completes successfully, then a sign-out event is recorded for the account.

**FEAT-21.SPEC-006-AC-09:** Given Nadia has no other active sessions and the "Sign out other devices" button is therefore not shown on FEAT-21.SPEC-003, then this automation is never triggered from that state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (signed out, no-action, failure) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
