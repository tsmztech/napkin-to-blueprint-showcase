---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-31.SPEC-005
spec_name: Support Access Authorization & Read-Only Rules
spec_slug: support-access-authorization-read-only-rules
parent_feature: FEAT-31
parent_feature_name: Operator Support Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 36
acceptance_criteria_count: 24
---

# Logic/Rule Spec: Support Access Authorization & Read-Only Rules

## Overview

**Name:** Support Access Authorization & Read-Only Rules
**ID:** FEAT-31.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may open, view, or never see a support session, the one-account-at-a-time and unconditional read-only constraints, the excluded actions (file downloads, data/accounting export generation, signing in as a client contact), and Nadia's standing right to see every session on her account.
**Parent Feature:** FEAT-31 -- Operator Support Access
**Governed Entity:** Support Access Session

## Scope and Non-Goals

**In Scope:**
- Field validation for every Support Access Session field
- The Dana-only, one-account-at-a-time authorization rule for opening a session
- The unconditional read-only rule and its excluded actions (edit, send, approve, pay, file-download, data/accounting export generation, client-contact sign-in)
- Nadia's standing right to see every session on her own account
- Owen and Priya's total exclusion from any visibility of support sessions
- Defaults and derivations for `operator`, `opened_at`, and `closed_at`

**Non-Goals:**
- The queue display, the mirrored screen layout, and the permanent banner -- owned by FEAT-31.SPEC-002 (Operator Support Session Console), which references this spec for the rules it displays.
- The step-by-step processing of opening and closing a session -- owned by FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) and FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity), which enforce these rules but are not their source of truth.
- A manual or on-demand close rule -- not established by any Stage 2 field; product-features.md's Validation & Limits states only that "a session ends automatically after a period of inactivity," so no manual-close authorization rule exists here, per the Brief's Non-Goals.
- Rules governing the freelancer's own data inside the mirrored view (e.g., what fields a proposal or invoice has) -- each underlying feature's own Logic/Rule spec owns its own entity's rules; this spec governs only the Support Access Session entity and the read-only constraint it imposes on top of them.

## Governed Entity

**Entity:** Support Access Session
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| freelancer_account | text (reference) | The one Freelancer Account being viewed or requested about (required) |
| request_text | text | Nadia's description of her problem, captured when she submits the request (required) |
| operator | text (reference) | Dana's identity, set only once she opens the session (required once opened) |
| opened_at | date/time | The timestamp the session opened (required once opened) |
| closed_at | date/time | The timestamp the session closed (required once closed) |

The entity has no explicit `status` field -- its lifecycle state (requested / opened / closed) is derived from which of `operator`, `opened_at`, and `closed_at` are set, per the Feature Dependency Map.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-31.SPEC-001 | Contact Support Screen | `request_text` field validation, on blur and on submit |
| FEAT-31.SPEC-002 | Operator Support Session Console | Authorization on "Open Session" tap; read-only enforcement rendered on every mirrored screen; screen entry restricted to Dana only |
| FEAT-31.SPEC-003 | Support Session Open & Read-Only Enforcement | Authorization check at open time; applies the excluded-action list for the session's duration |
| FEAT-31.SPEC-004 | Support Session Auto-Close on Inactivity | The single automatic close path; no manual-close rule exists to enforce |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| freelancer_account | Required; system-set to the account of the freelancer submitting the request, never user-entered | Always | On create | N/A -- system-set, never invalid | No |
| request_text | Required, non-empty, 1-2000 characters | Always | On blur and on submit | "Describe your problem before sending." / "Please shorten your message to 2000 characters or fewer." | Yes |
| operator | No validation beyond data type -- system-set only when Dana opens the session (FEAT-31.SPEC-003); never user-entered | Always | -- | -- | -- |
| opened_at | No validation beyond data type -- system-set timestamp on open (FEAT-31.SPEC-003) | Always | -- | -- | -- |
| closed_at | No validation beyond data type -- system-set timestamp on automatic close (FEAT-31.SPEC-004) | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Opening requires both fields together | operator, opened_at | `opened_at` is non-null if and only if `operator` is non-null -- both are written atomically by FEAT-31.SPEC-003 | Not applicable -- this is a system-write invariant with no user-facing validation path; a load or authorization failure leaves both fields unset together (FEAT-31.SPEC-003) |
| Closing requires opening first | opened_at, closed_at | `closed_at` can only be set on a record where `opened_at` is already set -- a session cannot close before it has opened | Not applicable -- system-write invariant; FEAT-31.SPEC-004 only ever evaluates records with `opened_at` set and `closed_at` unset |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Submit a support request | Nadia | Always, for her own account only | -- |
| Submit a support request | Owen, Priya, Dana | Never | The Contact Support screen (FEAT-31.SPEC-001) is not shown anywhere in their product view |
| View own support sessions in the activity trail | Nadia | Always -- every session on her account, per the Access Matrix's "View (sees every support session on her account)" | -- |
| View support sessions | Owen | Never | Support session entries never appear in his view of the activity trail; Access Matrix gives him "None" for Support Access |
| View support sessions | Priya | Never | Same as Owen -- "None" for Support Access |
| View the queue of pending requests | Dana | Always | -- |
| View the queue of pending requests | Nadia, Owen, Priya | Never | The Operator Support Session Console (FEAT-31.SPEC-002) does not exist anywhere in their product |
| Open a session | Dana | Always, provided she has no other session currently open (one-account-at-a-time) | If Dana already has an open session: blocking message "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." The open session is unaffected; Dana can Resume it or wait for the automatic close, after which she can open another |
| Open a session | Nadia, Owen, Priya | Never | No control to open a session exists anywhere for these roles |
| View the named freelancer's account data during an open session | Dana | Always, for the exact duration of her open session, read-only | -- |
| View the named freelancer's account data | Owen, Priya | Never | Not applicable -- these roles have no relationship to a support session and no such view is ever offered to them |
| Edit, send, approve, or pay any control during an open session | Dana | Never | Control is not shown as available (FEAT-31.SPEC-003); a direct attempt is refused: "Not available in a support session." |
| Download a deliverable file during an open session | Dana | Never | Download control is never offered; a direct attempt is refused: "Downloads are not available in a support session." |
| Generate a data or accounting export during an open session | Dana | Never | Export control is never offered; a direct attempt is refused: "Exports are not available in a support session." |
| Sign in as a client contact | Dana | Never | No such control exists anywhere in the product for this role |
| View Nadia's sign-in credentials | Dana | Never | Never displayed anywhere in the mirrored view |
| View the payment processor account reference | Dana | Never | Only connection status is ever shown; the reference itself is never displayed (feature-dependency-map.md, Payment Account Connection Data Sensitivity note) |
| Manually close an open session | Dana | Never -- no such action exists in the product | Not applicable -- no control exists to attempt; the sole close path is automatic on inactivity (FEAT-31.SPEC-004) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| operator | Unset (null) at creation; derived to Dana's identity the instant a session opens (FEAT-31.SPEC-003) | On open only | No -- system-set only |
| opened_at | Unset (null) at creation; derived to the current timestamp the instant a session opens (FEAT-31.SPEC-003) | On open only | No -- system-set only |
| closed_at | Unset (null) until the automatic inactivity close fires (FEAT-31.SPEC-004); derived to the current timestamp at that instant | On automatic close only | No -- system-set only; no manual close exists to override it with |

## Business Rules

- One session may be open per operator (Dana) at a time (XBR-29): opening a second while one is open is refused, per Authorization Rules above.
- Sessions are unconditionally read-only for their entire duration, with no exception -- excluded actions are edit, send, approve, pay, file-download, and data/accounting export generation (product-features.md, Validation & Limits; scope-boundaries.md SC-04).
- Dana never signs in as a client contact, under any circumstance (product-features.md, Primary Flows & Alternates; scope-boundaries.md SC-04).
- Nadia has a standing, unconditional right to see every session on her own account, in her activity trail (Access Matrix).
- Owen and Priya have no access whatsoever to support sessions and are never notified about them (Access Matrix; product-features.md, Access field: "Owen and Priya have no access and never see support sessions").
- No manual or on-demand close path exists for Dana; the only closure is the automatic inactivity close (FEAT-31.SPEC-004), per the Brief's Non-Goals.
- While one session is open, Dana's only permitted paths are to Resume it (or keep working in it) or to wait for the automatic inactivity close; any denied second-open message must name the open account, state that the session ends automatically after inactivity, and point to Resume -- it never instructs her to close the session, because no close control exists.
- Card data is never visible in a session because the product never holds it at all, in a session or otherwise (BRIEF.md, Constraints; product-features.md, Validation & Limits).

## Edge Cases

- **Dana attempts to construct a direct link to a file or export, bypassing the rendered console UI** -- The download or export is refused with the same exact denied message ("Downloads are not available in a support session." / "Exports are not available in a support session.") regardless of how the request is made, since these rules are enforced at the authorization boundary, not only in the rendered controls.
- **Dana's authorization is checked at the moment she selects "Open Session," not retained from an earlier page load** -- If she already has another session open in a separate tab, the second open attempt is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it.", even though nothing visibly changed on the screen she is looking at.
- **A support request exists for a freelancer account that has since begun deletion (FEAT-24)** -- Dana cannot open a session on it; the request is removed from the queue as part of the account's full deletion, per FEAT-31.SPEC-003's load-failure handling.
- **request_text at exactly 2000 characters** -- Passes validation. 2001 characters shows the length error.
- **request_text as whitespace only** -- Treated as empty; the "Describe your problem before sending." error applies.
- **Owen or Priya somehow reaches a stale reference to a support session** -- Treated as unauthorized like any out-of-scope access: the session and its content are never shown, consistent with their "None" Access Matrix entry.
- **A session's `opened_at` and `closed_at` are both set, and someone attempts to re-open it** -- Not possible: FEAT-31.SPEC-003 only ever operates on pending (unopened) requests from the queue; a closed session is never re-offered as an "Open Session" target.

## Acceptance Criteria

**FEAT-31.SPEC-005-AC-01:** Given Nadia is submitting a support request, when she leaves the problem description empty and attempts to send, then she sees "Describe your problem before sending." and the send does not proceed.

**FEAT-31.SPEC-005-AC-02:** Given Nadia enters a problem description of exactly 2000 characters, when she sends it, then it is accepted; at 2001 characters, she sees "Please shorten your message to 2000 characters or fewer."

**FEAT-31.SPEC-005-AC-03:** Given a session is opened by FEAT-31.SPEC-003, when the record is written, then `operator` and `opened_at` are both set together -- neither is ever set without the other.

**FEAT-31.SPEC-005-AC-04:** Given a session's `opened_at` is unset, then FEAT-31.SPEC-004's auto-close evaluation never considers it, since `closed_at` can only be set on a record where `opened_at` is already set.

**FEAT-31.SPEC-005-AC-05:** Given Nadia is signed in to her own account, when she opens her activity trail, then she sees every support session on her account, past and present.

**FEAT-31.SPEC-005-AC-06:** Given Owen or Priya is signed in to their client portal, when they look anywhere in their product view, then no support session content or entry is ever shown to them.

**FEAT-31.SPEC-005-AC-07:** Given Dana is signed in as the operator, when she opens the Support Session Console, then she sees the queue of pending requests.

**FEAT-31.SPEC-005-AC-08:** Given Nadia, Owen, or Priya attempts to reach the Support Session Console, then it does not exist anywhere in their product.

**FEAT-31.SPEC-005-AC-09:** Given Dana has no session currently open, when she opens one on a queued request, then the open succeeds.

**FEAT-31.SPEC-005-AC-10:** Given Dana already has a session open, when she attempts to open a second, then she sees "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." with no instruction to close anything, the open session stays open, and the second session does not open.

**FEAT-31.SPEC-005-AC-11:** Given Dana is inside an open session, when she views the freelancer's account data, then she sees it exactly as the freelancer would, for the duration of the session only.

**FEAT-31.SPEC-005-AC-12:** Given Dana is inside an open session, when she attempts any edit, send, approve, or pay action, then the control is not shown as available, and a direct attempt shows "Not available in a support session."

**FEAT-31.SPEC-005-AC-13:** Given Dana is inside an open session, when she attempts to download a deliverable file, then she sees "Downloads are not available in a support session."

**FEAT-31.SPEC-005-AC-14:** Given Dana is inside an open session, when she attempts to generate a data or accounting export, then she sees "Exports are not available in a support session."

**FEAT-31.SPEC-005-AC-15:** Given Dana is diagnosing a client-side sign-in problem, when she looks for a way to sign in as the client contact, then no such control exists anywhere in the product.

**FEAT-31.SPEC-005-AC-16:** Given Dana is inside an open session viewing account settings, when she looks for Nadia's sign-in credentials, then they are never displayed.

**FEAT-31.SPEC-005-AC-17:** Given Dana is inside an open session viewing payment settings, when she looks for the processor account reference, then only the connection status is shown, never the reference itself.

**FEAT-31.SPEC-005-AC-18:** Given Dana is inside an open session, when she looks for a way to end it manually, then no such control exists anywhere -- the session remains open until it closes automatically on inactivity.

**FEAT-31.SPEC-005-AC-19:** Given Dana attempts to construct a direct link to a file bypassing the rendered console, when the request reaches the authorization boundary, then it is refused with the same exact denied message as the rendered control.

**FEAT-31.SPEC-005-AC-20:** Given a queued request's freelancer account begins deletion (FEAT-24) before Dana opens it, when she attempts to open it, then the request has already been removed from the queue as part of that deletion.

**FEAT-31.SPEC-005-AC-21:** Given Nadia enters only whitespace in the problem description, when she attempts to send, then she sees "Describe your problem before sending." exactly as if the field were empty.

**FEAT-31.SPEC-005-AC-22:** Given no card data of any kind exists in the product, when Dana views any screen inside a session, then no card data is ever shown, because none is ever held.

**FEAT-31.SPEC-005-AC-23:** Given Dana has one session open in one browser tab, when she attempts to open a different request in a second tab, then the second attempt is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." even though the first tab's screen shows no visible change.

**FEAT-31.SPEC-005-AC-24:** Given a session has already closed (`closed_at` set), when anyone looks for a way to re-open it from the queue, then it is not offered there -- it was already removed from the pending queue the moment it opened, and closing does not return it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 18 | 18 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 8 | 8 |
| Edge Cases | 7 | 7 |
