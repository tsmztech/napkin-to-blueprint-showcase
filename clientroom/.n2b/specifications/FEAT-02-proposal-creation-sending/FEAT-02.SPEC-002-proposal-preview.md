---
document_type: spec
spec_type: screen
spec_id: FEAT-02.SPEC-002
spec_name: Proposal Preview
spec_slug: proposal-preview
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Proposal Preview

## Overview

**Name:** Proposal Preview
**ID:** FEAT-02.SPEC-002
**Type:** Screen
**Purpose:** Nadia reviews a branded, client-facing rendering of the current draft's scope, price, and payment schedule before sending it to the client.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- A read-only, branded rendering of the draft's content exactly as Owen will see it
- Initiating the Send action from this screen
- Returning to the editor to make further changes before sending

**Non-Goals:**
- Editing content on this screen -- editing happens only in FEAT-02.SPEC-001 (Proposal Draft Editor); this screen is read-only by design so Nadia reviews the exact content that will be sent.
- Performing the send itself -- the send transition, its validation, and its side effects are owned by FEAT-02.SPEC-010 (validation) and FEAT-02.SPEC-005 (Proposal Send); this screen only initiates that flow.
- Rendering the sent email -- the email's own layout and copy are owned by FEAT-02.SPEC-011 (Proposal Sent/Resent Email); this screen previews the in-portal proposal content, not the email.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Nadia taps "Preview" after entering valid scope and price | The Draft's current scope_description, price, currency, and payment_schedule_reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Send the proposal, return to edit | -- |
| Owen (Client Primary Contact) | No | No | This screen is Nadia's own pre-send review; no route into it exists from the client portal. Once sent, Owen reviews the proposal through his own portal view (FEAT-03), not this screen. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal route to this screen. |
| Dana (Support Operator) | Full screen, read-only (Send control not shown) | None | Reached only inside a logged, read-only support session (FEAT-31). |
| Unauthenticated | No | No | Redirected to freelancer sign-in. |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." The Draft that was being previewed remains saved and is reachable again from FEAT-02.SPEC-003 after re-authentication. |

## Layout and Content

**Header:** Screen title "Preview" with a back arrow (returns to FEAT-02.SPEC-001, editor pre-filled with the same content) at the left, and a "Send" primary action button at the right.

**Body:** A single-column, branded rendering styled with the freelancer's Branding Profile (logo and brand colour, FEAT-19, per XBR-31; falls back to a neutral default when unset), showing, in order:
- Freelancer's logo and brand colour applied to the page header band
- Project name and client name
- Scope Description, rendered as formatted text
- Price and Currency, prominently displayed
- Payment Schedule summary (structure and amounts, from FEAT-04), or "No payment schedule set yet" if none exists

**Footer:** None -- Send is in the header.

### Responsive Behavior

- **Compact:** Single-column rendering, full width; header condenses to title, back arrow, and Send button only.
- **Medium size class and above:** Rendering remains single-column, capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-02.SPEC-001 (Proposal Draft Editor), pre-filled with the previewed content | Screen closes | Animated transition back to the editor |
| Send button | Tap | Re-validates via FEAT-02.SPEC-010 (send-eligibility rules, including the Primary-contact check) and, if eligible, triggers FEAT-02.SPEC-005 (Proposal Send) | Button shows loading state during send | Success: navigates to FEAT-02.SPEC-003 (Proposal Detail) showing Sent state, with confirmation "Proposal sent to {Primary Contact name}." Failure: error banner, content preserved |
| Send button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Send button (the rendered content is read via normal document order between them).
- **Send feedback:** The success confirmation and any send-eligibility error are announced to assistive technology when they appear.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | N/A -- not reachable | N/A -- Preview is only reachable from a Draft that has already passed field validation (FEAT-02.SPEC-001 requires valid scope and price before offering "Preview"), so an empty-content state is never possible here | N/A |
| Loading | Skeleton placeholders for the branded rendering (header band, scope block, price block, payment schedule block) | Screen opens from FEAT-02.SPEC-001, before the Branding Profile (FEAT-19), Payment Schedule (FEAT-04), and Client/Project names finish loading -- none of these are handed off by the Entry Point, so this screen fetches them itself on open | The fetch completes and the Loaded state renders |
| Loaded (default) | Full branded rendering of the draft's content | The Loading fetch completes successfully | Nadia navigates away or taps Send |
| Sending | Send button shows a loading spinner; content remains visible but read-only (already the default) | Nadia taps Send and validation passes | Send completes or fails |
| Send Blocked | Inline banner above the Send button naming the unmet send-eligibility condition (e.g., "This client has no Primary contact yet.") with a link to resolve it | Send-eligibility validation fails (FEAT-02.SPEC-010) | Nadia resolves the condition and returns, or navigates away |
| Error | Error banner at the top with a retry option; the draft's content is unchanged. Two causes share this appearance: (a) the initial Loading fetch fails ("Could not load the preview." banner, Send hidden until retried) or (b) the send operation itself fails after eligibility passed ("Could not send this proposal." banner, content still shown) | (a) The Loading fetch fails, or (b) the send operation fails after eligibility passed (e.g., connectivity lost mid-send) | Nadia taps Retry (re-runs the failed fetch or the failed send) or navigates back to the editor |
| Offline/Degraded | Banner: "You're offline -- sending requires a connection." Send is disabled; the preview content itself remains viewable from local state | Connectivity is lost while this screen is open | Connectivity returns; Send re-enables |

## Validation Rules

Validation governed by FEAT-02.SPEC-010 (Proposal Validation & Business Rules). The Send button re-runs send-eligibility checks (required fields, positive price, and the client Primary-contact requirement, XBR-07) at the moment of tap -- not only at the time the editor was left -- since eligibility can change between preview and send.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-02.SPEC-001 (Proposal Draft Editor) | -- |
| Successful send | FEAT-02.SPEC-003 (Proposal Detail) | -- |
| Send Blocked banner link (no Primary contact) | Client contact list | FEAT-18 (Client Contact Management & Roles) |

## Data Model

**Creates:** None -- this screen renders existing Draft content; it does not persist new data.
**Reads:** Proposal -- scope_description, price, currency, payment_schedule_reference (carried directly from the Entry Point's context, no fetch needed). Payment Schedule -- structure and amounts (FEAT-04); Branding Profile -- logo and brand colour (FEAT-19); Client and Project -- names for display: none of these three are handed off by the Entry Point, so the screen fetches them itself on open, which is what the Loading state covers.
**Updates:** None directly -- a successful Send hands the transition to FEAT-02.SPEC-005.
**Deletes:** None.

## Business Rules

- The rendering shown here must exactly match what FEAT-02.SPEC-011 (Proposal Sent/Resent Email) and Owen's portal view (FEAT-03) will present, per the Feature Breakdown Brief's Shared Context: preview and email both apply the freelancer's Branding Profile consistently (XBR-31).
- Send eligibility (required fields, positive price, one-active-proposal cap, client Primary-contact requirement per XBR-07) is fully governed by FEAT-02.SPEC-010 -- this screen never bypasses those checks even though the editor validated once already.
- Sending from this screen is the sole trigger for FEAT-02.SPEC-005 within this feature's happy path (the Draft editor never sends directly).

## Edge Cases

- **Nadia taps Send twice rapidly** -- The second tap is ignored while the first send is in progress (button in loading state).
- **The client's last Primary contact is removed between opening Preview and tapping Send** -- Send is blocked with the Send Blocked state: "This client has no Primary contact yet." and a link into FEAT-18 to add one; the Draft is unaffected.
- **Network failure during send** -- Error banner: "Could not send this proposal. Check your connection and try again." with a Retry button; the Draft's content is preserved and the proposal remains in Draft status (no partial Sent state is created).
- **Nadia navigates back to the editor and changes the price, then returns to Preview** -- The rendering reflects the updated content; there is no stale-preview state because Preview always reads the Draft's current saved values.
- **Fetching the Branding Profile, Payment Schedule, or Client/Project names fails on open** -- The Loading state resolves to the Error state ("Could not load the preview.") rather than rendering with any of the three missing; Send is not offered until the retry succeeds, since Send re-validation depends on data this screen has not yet confirmed it can display correctly.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Navigation (inbound/outbound) | Entry point from Preview tap; return destination via the back arrow |
| FEAT-02.SPEC-003 (Proposal Detail) | Navigation (outbound) | Destination after a successful send |
| FEAT-02.SPEC-005 (Proposal Send) | Triggers (outbound) | Send button, after eligibility passes, triggers the send automation |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Send-eligibility rules re-checked at the moment of Send |
| FEAT-19 (Freelancer Branding) | References (inbound) | Branding Profile applied to the rendering |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| proposal_preview_viewed | time since draft last saved (seconds) | Screen finishes loading | supports success-metrics.md: "Proposal Send Speed" |
| proposal_send_blocked | blocking reason (no_primary_contact / other) | Send-eligibility validation fails on Send tap | N/A -- no Stage 2 metric measures blocked sends; retained so the send-eligibility gate's real-world frequency is observable |

## Acceptance Criteria

**FEAT-02.SPEC-002-AC-01:** Given Nadia taps "Preview" from a valid Draft, when the Preview screen loads, then it shows the branded rendering of the scope, price, currency, and payment schedule exactly as saved on the Draft.

**FEAT-02.SPEC-002-AC-02:** Given Nadia is viewing the Preview and the client has a Primary contact, when she taps "Send", then FEAT-02.SPEC-005 is triggered and, on success, she is navigated to FEAT-02.SPEC-003 with the confirmation "Proposal sent to {Primary Contact name}."

**FEAT-02.SPEC-002-AC-03:** Given the client has no Primary contact, when Nadia taps "Send" from Preview, then the Send Blocked banner "This client has no Primary contact yet." appears with a link into FEAT-18, and no send occurs.

**FEAT-02.SPEC-002-AC-04:** Given Nadia taps the back arrow on Preview, then she is returned to FEAT-02.SPEC-001 with the same content still populated.

**FEAT-02.SPEC-002-AC-05:** Given Nadia taps "Send" and the operation fails due to a network error, then an error banner appears with a Retry option and the proposal remains in Draft status.

**FEAT-02.SPEC-002-AC-06:** Given Nadia loses connectivity while viewing Preview, then the banner "You're offline -- sending requires a connection." appears and Send is disabled.

**FEAT-02.SPEC-002-AC-07:** Given Dana (Support Operator) opens this screen inside a logged support session, when she views it, then the content renders read-only with no Send control shown.

**FEAT-02.SPEC-002-AC-08:** Given Nadia taps "Send" twice in rapid succession, then the second tap is ignored while the first send is in progress.

**FEAT-02.SPEC-002-AC-09:** Given Nadia returns to Preview after changing the price in the editor, when the screen reloads, then it shows the updated price, not the previously previewed value.

**FEAT-02.SPEC-002-AC-10:** Given Nadia taps "Preview" and the Branding Profile, Payment Schedule, and Client/Project names have not yet finished loading, when the screen opens, then skeleton placeholders appear until the fetch completes, after which the full branded rendering displays.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 7 (empty [N/A], loading, loaded, sending, send blocked, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
