---
document_type: spec
spec_type: screen
spec_id: FEAT-24.SPEC-002
spec_name: Account Deletion Screen
spec_slug: account-deletion-screen
parent_feature: FEAT-24
parent_feature_name: Data Export & Account Deletion
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 19
---

# Screen Spec: Account Deletion Screen

## Overview

**Name:** Account Deletion Screen
**ID:** FEAT-24.SPEC-002
**Type:** Screen
**Purpose:** Nadia reviews any open-item warnings and gives the explicit confirmation required to permanently delete her account and all its data.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- Displaying open-item warnings (unpaid invoices, pending approvals) evaluated by FEAT-24.SPEC-006
- Capturing Nadia's explicit, heavier-than-normal confirmation of permanent deletion
- Showing deletion progress and the outcome (success or a reverted, intact account on failure)

**Non-Goals:**
- Determining which open items warrant a warning or which records are legally retained -- owned entirely by FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules); this screen only displays that determination's output.
- Executing the cascading deletion itself -- owned by FEAT-24.SPEC-004 (Account Deletion Processing); this screen only captures confirmation and shows that automation's progress and outcome.
- Exporting data before deletion -- handled separately by FEAT-24.SPEC-001 (Data Export Screen); this screen does not repeat or require that step, consistent with the Brief's Primary Flows describing export and deletion as separate, independently initiated actions.
- Offering any undo or restore path after confirmation -- excluded per product-features.md's Validation & Limits field ("Account deletion requires explicit confirmation given its irreversibility"); no cancel action exists once the cascade begins.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-24.SPEC-001 (Data Export Screen) | Nadia taps "Continue to close your account" | None -- this screen loads and evaluates open items and retention determinations fresh via FEAT-24.SPEC-006 |
| FEAT-21.SPEC-001 (Account Profile) | Nadia taps "Close account" in the Settings navigation shell (alternate direct path, since the shell's link routes to FEAT-24 generally and the feature's default entry is FEAT-24.SPEC-001; this row documents that this screen is never itself the shell's direct destination) | N/A -- see Non-Goals: the shell always lands on FEAT-24.SPEC-001 first per the Brief's Default Entry |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Review warnings, confirm, and initiate permanent deletion | -- |
| Owen (Client Primary Contact) | No | No | No navigation path in the client portal reaches this screen; a direct link shows the same out-of-scope explanation used elsewhere (XBR-09) |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal surface exists for this feature |
| Dana (Support Operator) | No | No | Excluded entirely from every read-only support session (XBR-29; FEAT-24.SPEC-007); a direct link during a session shows "This isn't available during a support session" |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-12 (Freelancer Financial Dashboard), not this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Any in-progress confirmation state (the acknowledgment checkbox and typed confirmation text) is discarded rather than preserved -- resuming a partially confirmed deletion after re-authentication is treated as starting over, since this is a security-sensitive, irreversible action |

## Layout and Content

**Header:** Screen title "Delete Your Account" with a back arrow (returns to FEAT-24.SPEC-001, Data Export Screen). The shared Settings navigation shell is not shown on this screen -- unlike the other Settings-area screens, this one is presented as a focused, full-attention flow rather than one item among a navigable list, consistent with the Shared UI Pattern's "deliberately heavier" confirmation step.

**Body:** A single-column content area:
- Warning region (only shown when FEAT-24.SPEC-006 finds open items): up to two warning banners, the unpaid-invoice banner first and the pending-approval banner second, each rendered with the exact title and body text defined in FEAT-24.SPEC-006 (Business Rules, "Exact warning and notice text") with its count and amount placeholders filled, and neither blocking progress.
- Changed-warnings notice (only shown in the Loaded -- warnings changed state): a line directly above the warning region reading "Your open items changed. Review the updated warnings, then tap Delete My Account again."
- Retention notice: always shown, rendered with the exact text defined in FEAT-24.SPEC-006 (Business Rules, "Exact warning and notice text"), with the retention period filled from platform parameter: `financial-record-legal-retention-period`.
- Confirmation region, positioned below the warnings and retention notice:
  - An acknowledgment checkbox: "I understand this permanently deletes my account and all its data, and this cannot be undone."
  - A confirmation text input, labeled "Type DELETE to confirm."
  - A "Delete My Account" button, disabled until both the checkbox is checked and the typed text exactly matches "DELETE."
  - A "Cancel" button/link, always enabled, returning to FEAT-24.SPEC-001.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described above, full width; warning banners stack vertically; the Delete My Account and Cancel buttons stack full-width, Delete My Account above Cancel.
- **Medium size class and above:** Content is capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping. Delete My Account and Cancel sit side by side, Cancel to the left.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-24.SPEC-001 (Data Export Screen) | Screen closes | Standard navigation transition |
| Acknowledgment checkbox | Tap | Toggles acknowledgment | Delete My Account re-evaluates its enabled condition | Checkbox shows checked/unchecked state |
| Confirmation text input | Type | Captures typed text | Delete My Account re-evaluates its enabled condition on each keystroke | Standard input focus and text-entry state |
| "Delete My Account" button | Tap (enabled only when checkbox is checked AND input exactly equals "DELETE") | 1. Re-runs FEAT-24.SPEC-006 and compares the result with the warnings currently displayed (open_unpaid_invoice_present, open_invoice_count, open_invoice_totals, pending_approval_present, pending_approval_count). 2a. If identical: triggers FEAT-24.SPEC-009 (Account Deletion Final Warning Notification), then FEAT-24.SPEC-004 (Account Deletion Processing). 2b. If any value differs: the tap is rejected -- neither FEAT-24.SPEC-009 nor FEAT-24.SPEC-004 is triggered | 2a: Screen enters Processing state. 2b: Screen enters Loaded -- warnings changed state | 2a: Button shows loading state; screen shows "Deleting your account -- this can take a few minutes for accounts with a lot of history." 2b: Warning region shows the current banners (or none), the changed-warnings notice appears, and focus moves to the notice; the checkbox and typed text are kept |
| "Cancel" button/link | Tap | Navigate to FEAT-24.SPEC-001 (Data Export Screen); no deletion action taken | Screen closes | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Back arrow -> warning banners (if present) -> retention notice -> acknowledgment checkbox -> confirmation text input -> Delete My Account -> Cancel.
- **Changed-warnings announcement:** The changed-warnings notice and the refreshed banner text are announced when the tap is rejected, and focus moves to the notice.
- **Warning announcements:** Each warning banner's text is announced to assistive technology when the screen loads with open items present.
- **Confirmation-state announcements:** The Delete My Account button's enabled/disabled transition is announced when the checkbox and typed text together satisfy or stop satisfying the confirmation condition.
- **Processing announcements:** The "Deleting your account..." progress message is announced when processing begins.
- **Keyboard alternatives:** Every action (checkbox, text input, Delete My Account, Cancel, back arrow) is a standard keyboard-operable control; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A loading indicator is shown in place of the warning region and confirmation controls while FEAT-24.SPEC-006 evaluates open items and retention determinations | Screen first opens | Evaluation completes |
| Loaded -- no warnings | No warning banners; retention notice and confirmation region shown directly | FEAT-24.SPEC-006 finds no open items | Nadia confirms or cancels |
| Loaded -- with warnings | One or more warning banners shown above the retention notice and confirmation region | FEAT-24.SPEC-006 finds one or more open items | Nadia confirms or cancels |
| Confirming | Delete My Account button enabled | Checkbox checked AND typed text exactly "DELETE" | Nadia taps Delete My Account (re-check identical: Processing; re-check differs: Loaded -- warnings changed), or un-checks/edits the text (returns to Loaded) |
| Loaded -- warnings changed | The current warning banners (or none, if every open item has since resolved) are shown, with the changed-warnings notice above them. Checkbox stays checked, typed text is kept, and Delete My Account stays enabled; nothing has been sent to FEAT-24.SPEC-009 or FEAT-24.SPEC-004 | Delete My Account tapped and the FEAT-24.SPEC-006 re-check differs from the warnings displayed | Nadia taps Delete My Account again (re-check against the now-displayed warnings: Processing if identical, this state again if they changed once more), un-checks/edits the text (returns to Loaded), or cancels |
| Processing | Progress indicator and "Deleting your account -- this can take a few minutes for accounts with a lot of history." All controls disabled | Delete My Account tapped and the re-check is identical to the displayed warnings | FEAT-24.SPEC-004 reports completion (its commit report, after which Nadia is signed out) or a pre-commit failure |
| Error (deletion failed) | Error banner: "We couldn't complete account deletion. Your account has not been changed. Try again." Confirmation region re-enabled, checkbox and text cleared | FEAT-24.SPEC-004 reports failure and reverts the account to Active | Nadia re-confirms |
| Offline/Degraded | Banner: "You're offline. Reconnect to continue." Confirmation region disabled; any evaluation or processing already in flight is unaffected server-side | Connectivity lost while this screen is open | Connectivity restored -- the screen re-fetches current state |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Confirmation text input | Must exactly match "DELETE" (case-sensitive) | On change (continuously re-evaluated) | No error message shown -- the Delete My Account button simply stays disabled until the text matches exactly |

Open-item warning content (exact banner text and placeholders), the retention-notice text, and retention classification are governed by FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules); this screen only displays that spec's output verbatim.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-24.SPEC-001 (Data Export Screen) | -- |
| Cancel button/link | FEAT-24.SPEC-001 (Data Export Screen) | -- |
| Deletion completes | None -- Nadia is signed out; there is no screen to return to since her account no longer exists | -- |

## Data Model

**Creates:** None directly -- Delete My Account triggers FEAT-24.SPEC-009 and FEAT-24.SPEC-004, which perform the actual notification and deletion.
**Reads:** Freelancer Account and its open-item indicators (unpaid Invoice status, pending-approval Milestone status) and retention classification, all via FEAT-24.SPEC-006.
**Updates:** None directly.
**Deletes:** None directly -- the cascade is entirely FEAT-24.SPEC-004's responsibility.

## Business Rules

- Deletion requires explicit confirmation -- the acknowledgment checkbox AND an exact-match typed "DELETE" -- before the Delete My Account button is enabled (product-features.md, Validation & Limits field).
- Open items warn without blocking: FEAT-24.SPEC-006's warnings are informational only and never disable the confirmation controls (product-features.md, Primary Flows & Alternates).
- The re-check at tap time gates everything: FEAT-24.SPEC-009 and FEAT-24.SPEC-004 fire only after the re-check confirms the warnings Nadia is looking at are current. A changed picture rejects the tap and requires a fresh tap.
- FEAT-24.SPEC-009 (final warning email) fires at the moment of confirmation (an accepted tap), not after FEAT-24.SPEC-004 completes -- it is Nadia's own record of exactly when she confirmed, independent of how long processing takes.
- A failed deletion (FEAT-24.SPEC-004 outcome) always leaves the account fully intact and re-enables the confirmation controls -- there is no partially deleted state ever shown (product-features.md, States field).
- FEAT-24.SPEC-007 confines this entire screen, and every action on it, to Nadia and her own account.

## Edge Cases

- **Open items change between screen load and confirmation (e.g., Nadia sends a new invoice in another tab, then returns here and confirms)** -- The confirmation re-checks open items and retention determinations at the moment Delete My Account is tapped, per FEAT-24.SPEC-006; if any compared value differs from what is displayed (including a warning that has disappeared), the tap is rejected: the screen moves to Loaded -- warnings changed, shows the refreshed banners and the changed-warnings notice, and fires neither FEAT-24.SPEC-009 nor FEAT-24.SPEC-004. Nadia must tap Delete My Account again to proceed, and that second tap is compared against the now-displayed warnings. Resolution: reject-with-refresh -- the checkbox and typed text are kept, so only the fresh tap is needed. A re-check that cannot complete (offline or evaluation error) also rejects the tap and shows the Offline/Degraded banner or "We couldn't check your open items. Try again."; nothing is triggered.
- **Nadia navigates away while Processing is underway** -- The deletion cascade (FEAT-24.SPEC-004) continues server-side regardless of whether this screen remains open; if she returns to any product screen before it completes, she is signed out once it finishes.
- **Nadia double-taps Delete My Account** -- The second tap is ignored while the button is in its loading/disabled state during Processing.
- **The typed confirmation text is pasted rather than typed** -- Treated identically to typed input; the exact-match check applies the same way.
- **Nadia edits the confirmation text after Delete My Account is already Processing** -- Not possible; all controls are disabled during Processing.
- **A retained financial record's classification changes between the warning display and the actual cascade (e.g., an invoice is paid in the moments after the warning was shown)** -- FEAT-24.SPEC-004 re-evaluates retention classification at cascade execution time via FEAT-24.SPEC-006, independent of what this screen displayed at load; the displayed warning reflects the state at evaluation time and is not guaranteed to be identical to the state at the exact moment the cascade runs, since a warning is advisory, not a lock.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules) | References (inbound) | Supplies the open-item warning content and retention classification this screen displays |
| FEAT-24.SPEC-009 (Account Deletion Final Warning Notification) | Triggers (outbound) | Confirmation fires this notification |
| FEAT-24.SPEC-004 (Account Deletion Processing) | Triggers (outbound) | Confirmation starts the cascading deletion; this screen shows its progress and outcome |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Governs who can reach and act on this screen |
| FEAT-24.SPEC-001 (Data Export Screen) | Navigation (inbound / outbound) | Entry point via "Continue to close your account"; Cancel and the back arrow return here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| account_deletion_requested | open_items_present (yes/no) | Nadia opens this screen and FEAT-24.SPEC-006's evaluation completes | N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; retained because product-features.md's Signals field names this event explicitly |
| account_deletion_confirmed | open_items_present (yes/no) | Nadia taps Delete My Account with a valid confirmation | N/A -- same reason as above; retained for observability of the confirmation moment distinct from the eventual completion signal emitted by FEAT-24.SPEC-004 |

## Acceptance Criteria

**FEAT-24.SPEC-002-AC-01:** Given Nadia is on FEAT-24.SPEC-001 and taps "Continue to close your account," when this screen loads, then it evaluates open items and retention determinations via FEAT-24.SPEC-006 before showing any warning or confirmation content.

**FEAT-24.SPEC-002-AC-02:** Given Nadia has an unpaid invoice, when this screen finishes loading, then a warning banner describes that specific consequence without disabling the confirmation controls.

**FEAT-24.SPEC-002-AC-03:** Given Nadia has no open items, when this screen finishes loading, then no warning banners appear and only the retention notice and confirmation region are shown.

**FEAT-24.SPEC-002-AC-04:** Given Nadia checks the acknowledgment checkbox but leaves the confirmation text empty, when she looks at "Delete My Account," then it remains disabled.

**FEAT-24.SPEC-002-AC-05:** Given Nadia checks the acknowledgment checkbox and types "delete" (lowercase), when she looks at "Delete My Account," then it remains disabled, since the match is case-sensitive.

**FEAT-24.SPEC-002-AC-06:** Given Nadia checks the acknowledgment checkbox and types "DELETE" exactly, when she looks at "Delete My Account," then it becomes enabled.

**FEAT-24.SPEC-002-AC-07:** Given Nadia has satisfied both confirmation conditions, when she taps "Delete My Account," then FEAT-24.SPEC-009 fires immediately and FEAT-24.SPEC-004 begins, and the screen shows "Deleting your account -- this can take a few minutes for accounts with a lot of history."

**FEAT-24.SPEC-002-AC-08:** Given deletion processing is underway, when Nadia looks at the screen, then every control is disabled and a second tap on Delete My Account has no effect.

**FEAT-24.SPEC-002-AC-09:** Given FEAT-24.SPEC-004 reports successful completion, when processing finishes, then Nadia is signed out and no screen in the product remains for her account.

**FEAT-24.SPEC-002-AC-10:** Given FEAT-24.SPEC-004 reports failure, when processing ends, then the screen shows "We couldn't complete account deletion. Your account has not been changed. Try again." with the checkbox and confirmation text cleared.

**FEAT-24.SPEC-002-AC-11:** Given Nadia taps Cancel at any point before confirming, when she does so, then no deletion action is taken and she returns to FEAT-24.SPEC-001.

**FEAT-24.SPEC-002-AC-12:** Given Owen (Client Primary Contact) has no navigation path to this screen, when he follows any link in the client portal, then he never reaches it.

**FEAT-24.SPEC-002-AC-13:** Given Dana (Support Operator) is in an active read-only support session, when she attempts to open this screen directly, then she sees "This isn't available during a support session."

**FEAT-24.SPEC-002-AC-14:** Given Nadia's session expires while she has partially filled the confirmation region, when she is shown the expired-session dialog and signs back in, then the confirmation region is reset (checkbox unchecked, text cleared) rather than restored.

**FEAT-24.SPEC-002-AC-15:** Given Nadia sends a new unpaid invoice in another tab after this screen has already loaded, when she taps "Delete My Account" here, then the open-item evaluation re-runs at that moment, the tap is rejected, the screen shows the new unpaid-invoice banner and the changed-warnings notice, and neither FEAT-24.SPEC-009 nor FEAT-24.SPEC-004 has fired.

**FEAT-24.SPEC-002-AC-16:** Given Nadia loses connectivity while reviewing this screen, when the connection drops, then the banner "You're offline. Reconnect to continue." appears and the confirmation region is disabled.

**FEAT-24.SPEC-002-AC-17:** Given the screen is in Loaded -- warnings changed after a rejected tap and the checkbox and text are still valid, when Nadia taps "Delete My Account" again and the re-check matches the warnings now displayed, then FEAT-24.SPEC-009 fires, FEAT-24.SPEC-004 begins, and the screen enters Processing.

**FEAT-24.SPEC-002-AC-18:** Given Nadia has 1 unpaid invoice of $1,200.00 displayed and pays it in another tab, when she taps "Delete My Account," then the tap is rejected, the unpaid-invoice banner is removed, the notice "Your open items changed. Review the updated warnings, then tap Delete My Account again." appears, and no notification or deletion is triggered.

**FEAT-24.SPEC-002-AC-19:** Given Nadia has an open invoice and a pending approval, when this screen loads, then the unpaid-invoice banner appears above the pending-approval banner and above the retention notice, each with the exact text defined in FEAT-24.SPEC-006.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 8 (loading, loaded-no-warnings, loaded-with-warnings, loaded-warnings-changed, confirming, processing, error, offline) | 8 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
