---
document_type: spec
spec_type: screen
spec_id: FEAT-31.SPEC-002
spec_name: Operator Support Session Console
spec_slug: operator-support-session-console
parent_feature: FEAT-31
parent_feature_name: Operator Support Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Operator Support Session Console

## Overview

**Name:** Operator Support Session Console
**ID:** FEAT-31.SPEC-002
**Type:** Screen
**Purpose:** Dana sees the queue of open support requests, opens a read-only session on one named freelancer's account, works through that freelancer's own screens under a permanent read-only banner, and steps away when done -- the session then ends on its own after inactivity.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- The queue of pending (unopened) support requests, oldest first, across every freelancer account
- Opening a read-only session on one named freelancer account at a time
- Hosting the mirrored, read-only view of that account's own screens (FEAT-01 through FEAT-25) under the permanent "Read-only support session" banner
- Returning Dana to the queue when a session closes, whether by inactivity (FEAT-31.SPEC-004) or by her own navigation away

**Non-Goals:**
- A manual "End Session" or "Close" control -- not established by any Stage 2 field; product-features.md's Validation & Limits states only that "a session ends automatically after a period of inactivity," so this screen defines no on-demand close action of its own. Dana ends her diagnosis simply by stepping away; the only mechanism that actually closes the record is the automatic inactivity close (FEAT-31.SPEC-004).
- The content and controls of the mirrored screens themselves -- each underlying feature (FEAT-01 through FEAT-25) owns its own screen's layout and content; this spec owns only the queue, the session-open action, the permanent banner, and the read-only enforcement surface, per FEAT-31.SPEC-003.
- File downloads and data or accounting export generation from inside a session -- excluded per scope-boundaries.md (SC-04) and FEAT-31.SPEC-005; these controls are never offered on any mirrored screen during a session.
- Signing in as a client contact to diagnose a client-side problem -- excluded per product-features.md's Primary Flows & Alternates ("Dana never signs in as a client contact") and scope-boundaries.md (SC-04).

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External (Dana's own operator access, outside the freelancer- and client-facing product) | Dana opens the console | None -- queue loads fresh |
| FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity) | An open session closes automatically | Notice naming the freelancer whose session just closed |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Dana (Support Operator) | Full screen | Open a session on one queued request at a time; navigate the mirrored read-only view; return to the queue | -- |
| Nadia (Freelancer) | No | No | This screen does not exist anywhere in her product; she never reaches it |
| Owen (Client Primary Contact) | No | No | Same -- not reachable from the client portal |
| Priya (Client Reviewer Contact) | No | No | Same -- not reachable from the client portal |
| Unauthenticated | No | No | Redirected to Dana's own sign-in; this console is never reachable without her operator credentials |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- if a support session was open, it remains open in the background (subject to its own inactivity clock, FEAT-31.SPEC-004) and the mirrored view resumes once Dana signs back in |

## Layout and Content

**Header:** Screen title "Support Session Console." When no session is open: no further header content. When a session is open: a full-width, persistent banner reading "Read-only support session -- {freelancer_account name}" replaces the default header treatment and remains visible on every mirrored screen for the duration of the session.

**Body (queue view, no session open):** A list of pending Support Access Session requests, oldest first, each row showing: the freelancer's account name, a one-line preview of the request text, and the time the request was submitted. Each row carries an "Open Session" action. While the queue is being fetched, the body shows the text "Loading support requests..." in place of the list. If the fetch fails, the body shows the text "Support requests could not be loaded right now." with a "Retry" button beneath it.

**Body (session open):** The mirrored screen for whatever feature Dana is currently viewing (FEAT-01 through FEAT-25), rendered exactly as Nadia would see it, with every edit, send, approve, pay, file-download, and export-generation control disabled or not shown (FEAT-31.SPEC-003, FEAT-31.SPEC-005). A "Return to queue" link sits below the banner at all times.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Queue list is single-column, full width; the read-only banner spans the full width above the mirrored content and stays fixed at the top on scroll.
- **Medium size class and above:** Queue list remains single-column, capped at a consistent platform-wide list width; the banner remains full-width and fixed at the top; mirrored screens inherit each underlying feature's own responsive behavior unchanged.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Queue row "Open Session" | Tap | 1. Check authorization via FEAT-31.SPEC-005 (Dana has no other session open). 2. If authorized, trigger FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement). | Button shows loading state; on success the screen transitions from queue to mirrored view with the banner | Success: banner appears, mirrored view loads. Failure: exact denied or error message shown inline (see Edge Cases) |
| Queue row "Open Session" (while another session is already open) | Tap | No action taken -- blocked before FEAT-31.SPEC-003 is triggered | Control remains present but the attempt is refused; the open session stays open | Inline message on the row: "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." The persistent "Session open on {freelancer_account name} -- Resume" notice remains shown above the queue. Meanwhile Dana can tap Resume to keep working in the open session, or step away and wait for the automatic close, after which every row's "Open Session" works again |
| Queue "Retry" button (queue load-error state) | Tap | Re-fetches the queue of pending requests | Body returns to the queue Loading state, then to Queue populated, Empty queue, or (on repeated failure) the queue Load-error state again | "Loading support requests..." while fetching; on repeated failure the same "Support requests could not be loaded right now." text with "Retry" remains |
| Row "Retry" button (Error state, session cannot open) | Tap | Re-runs the "Open Session" action for that same request: authorization check (FEAT-31.SPEC-005), then FEAT-31.SPEC-003 | Row returns to the Opening state | Same feedback as "Open Session": banner and mirrored view on success; the row's inline reason and "Retry" button again on repeated failure |
| Mirrored screen edit/send/approve/pay/download/export controls | Tap (any) | Blocked before reaching the underlying feature's own logic, per FEAT-31.SPEC-005 | No state change | "Not available in a support session." (or, for downloads/exports specifically, "Downloads are not available in a support session." / "Exports are not available in a support session.") |
| "Return to queue" link | Tap | Navigates back to the queue view; the session itself is not closed by this action | Queue view shown | Queue list appears; if the session is still open, its row (if still pending elsewhere) is replaced by a persistent notice: "Session open on {freelancer_account name} -- Resume" |
| "Resume" notice (session still open) | Tap | Returns to the mirrored view at its last screen | Mirrored view with banner reappears | Same banner and mirrored screen as before navigating away |

### Accessibility Notes

- **Focus order:** Queue rows in submitted order (oldest first), each with its "Open Session" control; inside a session, the banner text is announced first, followed by the mirrored screen's own focus order, followed by "Return to queue."
- **Dynamic announcements:** The read-only banner is announced to assistive technology the instant a session opens (not just visually shown), and again if Dana resumes an open session from the queue. Every blocked-control message ("Not available in a support session.") is announced at the moment of the attempt, not only shown visually -- consistent with ASMP-27's requirement that the read-only state never relies on colour alone.
- **Keyboard alternatives:** Every action on this screen, including "Open Session," "Return to queue," and "Resume," is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Queue loading | "Loading support requests..." text in the body; no rows and no "Open Session" controls shown | Dana opens the console, returns to the queue, or taps the queue "Retry" button | The queue fetch succeeds (Empty queue or Queue populated) or fails (Queue load error) |
| Queue load error | "Support requests could not be loaded right now." text with a "Retry" button in the body; no rows shown; nothing about any request changes | The queue fetch fails | Dana taps "Retry" (returns to Queue loading) |
| Empty queue | "No open support requests." message in the body | No pending requests exist | A new request is submitted (FEAT-31.SPEC-001) |
| Queue populated | List of pending requests as described in Layout and Content | One or more pending requests exist | Dana opens one |
| Opening | Loading indicator over the queue row Dana selected | Dana taps "Open Session" and authorization passes | FEAT-31.SPEC-003 completes (success or failure) |
| Session open (mirrored view) | Permanent read-only banner plus the mirrored screen for whichever feature Dana is viewing; loads like Nadia's own screens (Non-Functional Notes, Responsiveness) | FEAT-31.SPEC-003 completes successfully | Session closes (FEAT-31.SPEC-004) |
| Queue (session open elsewhere) | Queue list shown with a persistent "Session open on {freelancer_account name} -- Resume" notice in place of that request's row | Dana taps "Return to queue" while a session remains open | Dana taps "Resume," or the session closes automatically (FEAT-31.SPEC-004) |
| Error (session cannot open) | Inline message on the queue row naming the specific reason (e.g., "This account's data could not be loaded right now.") with a "Retry" button on that row; nothing about the account changes | FEAT-31.SPEC-003 reports a load failure | Dana taps the row's "Retry" button (returns to Opening) or taps "Open Session" on a different request |
| Offline/Degraded | N/A -- product-features.md's States field marks this feature's Offline-degraded state "N/A -- support sessions are an operator-side, connectivity-required action" | -- | -- |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Authorization for opening a session, and every read-only enforcement rule applied to the mirrored view, is governed by FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules). This screen checks authorization the instant "Open Session" is tapped, and applies the read-only enforcement continuously for the session's duration.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Open Session (success) | The freelancer's own default landing screen (mirrored, read-only) | FEAT-01 through FEAT-25 (whichever the freelancer would land on) |
| Navigating within a session | Whichever mirrored screen Dana selects | FEAT-01 through FEAT-25 |
| Return to queue | This screen (queue view) | -- |
| Session auto-closes | This screen (queue view), with a notice | -- |

## Data Model

**Creates:** None (FEAT-31.SPEC-001 creates the Support Access Session record).
**Reads:** Support Access Session -- list of records with `operator` unset (the queue), and the single selected record's `freelancer_account` and `request_text` once opened. Freelancer Account and its dependent data across FEAT-01 through FEAT-25 -- read-only, exactly as Nadia would see it, for the duration of an open session.
**Updates:** None directly -- FEAT-31.SPEC-003 sets `operator` and `opened_at`, and FEAT-31.SPEC-004 sets `closed_at`; this screen triggers and displays those changes but does not write them itself.
**Deletes:** None.

## Business Rules

- Only one Support Access Session may be open at a time for Dana (one-account-at-a-time), per FEAT-31.SPEC-005 and XBR-29.
- Every control an edit, send, approve, pay, file-download, or export action would use is unavailable for the entire duration of an open session, with no exception, per FEAT-31.SPEC-003 and FEAT-31.SPEC-005.
- This screen provides no manual "End Session" control -- the only path that closes an open session is the automatic inactivity close (FEAT-31.SPEC-004), per the Brief's Non-Goals ("A manual or on-demand close control for Dana"). Dana ends her diagnosis simply by stepping away; the session record remains technically open, enforcing read-only, until inactivity closes it automatically.
- The queue lists requests across every freelancer account but never merges more than one account's data into a single mirrored view -- each open session is scoped to exactly one Freelancer Account (dependency map, Read (list) note).
- XBR-29: sessions are read-only in every feature, cover one account at a time, end after inactivity, exclude file downloads and data/accounting exports, are always announced to the freelancer by email (FEAT-31.SPEC-007), and are always listed in her trail (FEAT-13).

## Edge Cases

- **A session cannot open (the account's data cannot be loaded read-only)** -- Nothing about the account changes; Dana sees the specific reason inline on the queue row, per product-features.md's Error state definition.
- **Dana taps "Open Session" twice rapidly** -- Second tap is ignored while the first attempt is in progress (row shows loading state).
- **Dana attempts to open a second session while one is already open** -- Refused before FEAT-31.SPEC-003 is triggered, with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it."; nothing about either account changes. Dana can Resume the open session or wait for it to close automatically; no manual close exists.
- **The queue fetch fails or is slow** -- The body shows the queue Loading state, then on failure the queue Load-error state with "Retry"; an open session (if any) is unaffected and its "Resume" notice remains available.
- **A session Dana had open closes automatically (inactivity) while she is mid-navigation on a mirrored screen** -- She is returned to the queue view immediately, with a notice naming the freelancer and stating the session closed after inactivity; because the session is unconditionally read-only, no in-progress work is ever lost.
- **The freelancer account tied to a queued request is deleted (FEAT-24) before Dana opens it** -- The request disappears from the queue as part of that account's full removal; nothing is shown to Dana beyond the row no longer being present.
- **Concurrent-edit conflict** -- Not applicable to this screen directly: the dependency map's Contention note for Support Access Session states "None -- only Dana opens and closes a session, one account at a time, and the record is never edited after it closes; Nadia only reads it." The mirrored view's own underlying entities are read-only here (Dana never writes), so no load-then-save race exists on this screen for Dana to encounter; any contention among Nadia's own concurrent sessions is each underlying feature's own concern, unaffected by Dana's read-only presence.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-001 (Contact Support Screen) | Navigation (inbound, indirect) | A submitted request populates this screen's queue |
| FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | Triggers (outbound) | "Open Session" fires this automation |
| FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity) | Triggered by (inbound) | Auto-close returns Dana to the queue with a notice |
| FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules) | References (inbound) | Authorization and every read-only enforcement rule applied here |
| FEAT-32 (Payment Account Connection) | Navigation (outbound) | Inside a session, Dana sees only the payment connection status, never the processor account reference or credentials (feature-dependency-map.md, Cross-Feature Touchpoints) |
| FEAT-01 through FEAT-25 (various features) | Navigation (outbound) | The mirrored read-only view reads each feature's own screens and data exactly as Nadia would see them, with every write control disabled |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_session_open_attempted | outcome (opened / denied / load_failed) | Dana taps "Open Session" | N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained per product-features.md's Signals field so support activity stays observable |
| support_session_resumed | -- | Dana taps "Resume" on an open session from the queue | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-31.SPEC-002-AC-01:** Given Dana opens the Support Session Console with two pending requests, when the queue loads, then both appear oldest first, each with the freelancer's account name, a preview of the request text, and the submission time.

**FEAT-31.SPEC-002-AC-02:** Given Dana has no session open, when she taps "Open Session" on a queued request, then FEAT-31.SPEC-003 opens the session and the mirrored view appears under the "Read-only support session -- {freelancer_account name}" banner.

**FEAT-31.SPEC-002-AC-03:** Given Dana is inside an open session, when she looks at any edit, send, approve, or pay control on a mirrored screen, then it is disabled or not shown, and a direct attempt shows "Not available in a support session."

**FEAT-31.SPEC-002-AC-04:** Given Dana is inside an open session, when she attempts to download a deliverable file, then the control is not offered and a direct attempt shows "Downloads are not available in a support session."

**FEAT-31.SPEC-002-AC-05:** Given Dana already has a session open on one freelancer account, when she taps "Open Session" on a different queued request, then the attempt is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it.", the "Resume" notice remains shown, and no second session opens.

**FEAT-31.SPEC-002-AC-06:** Given Dana is inside an open session, when she taps "Return to queue," then the queue view appears with a persistent "Session open on {freelancer_account name} -- Resume" notice, and the session itself remains open.

**FEAT-31.SPEC-002-AC-07:** Given Dana's open session has just closed automatically after inactivity (FEAT-31.SPEC-004) while she was viewing a mirrored screen, then she is returned to the queue view immediately with a notice naming the freelancer and stating the session closed after inactivity.

**FEAT-31.SPEC-002-AC-08:** Given a queued request's account data cannot be loaded read-only, when Dana taps "Open Session" on it, then nothing about the account changes and she sees the specific reason inline on that row.

**FEAT-31.SPEC-002-AC-09:** Given Dana taps "Open Session" twice in rapid succession on the same request, when the first attempt is still in progress, then the second tap has no effect.

**FEAT-31.SPEC-002-AC-10:** Given no support requests are pending, when Dana opens the console, then it shows "No open support requests."

**FEAT-31.SPEC-002-AC-11:** Given Dana is inside a session viewing payment settings, when she looks for the payment connection detail, then she sees connection status only, never the processor account reference or credentials.

**FEAT-31.SPEC-002-AC-12:** Given Dana opens the console, when the queue fetch is in progress, then the body shows "Loading support requests..." with no rows and no "Open Session" controls.

**FEAT-31.SPEC-002-AC-13:** Given the queue fetch fails, when the console finishes loading, then the body shows "Support requests could not be loaded right now." with a "Retry" button; when Dana taps "Retry" and the fetch succeeds, then the queue (or "No open support requests.") appears.

**FEAT-31.SPEC-002-AC-14:** Given a session could not open and the row shows its inline reason with a "Retry" button, when Dana taps "Retry," then the row returns to its loading state and the open action runs again for that same request.

**FEAT-31.SPEC-002-AC-15:** Given Dana has a session open and has returned to the queue, when she taps "Open Session" on a different request, then she sees no manual close instruction, only the message naming the open account, stating it ends automatically after inactivity, and pointing to Resume, and tapping "Resume" returns her to the open session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 9 (queue loading, queue load error, empty queue, populated, opening, session open, queue with open session, error, offline/degraded N/A) | 9 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
