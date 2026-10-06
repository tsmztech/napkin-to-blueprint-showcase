---
document_type: spec
spec_type: screen
spec_id: FEAT-03.SPEC-001
spec_name: Proposal Review & Accept
spec_slug: proposal-review-accept
parent_feature: FEAT-03
parent_feature_name: Proposal Acceptance
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Proposal Review & Accept

## Overview

**Name:** Proposal Review & Accept
**ID:** FEAT-03.SPEC-001
**Type:** Screen
**Purpose:** Owen, the client's Primary Contact, reads a sent proposal's scope and price and either accepts it (recording a timestamped, permanent acceptance) or opens Request Changes to send Nadia a note instead.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- Rendering the sent proposal's scope description and price fully before the Accept control activates
- The Accept control and its confirmation
- The Request Changes control that opens FEAT-03.SPEC-002
- Displaying the permanent "Accepted on {date}" marker once acceptance is recorded
- Redirecting to the current proposal version when the opened version has been voided (edited-after-sending)
- Showing Priya's stage-only view and Dana's read-only support view of this same entry point

**Non-Goals:**
- A formal "decline" state -- excluded per the Feature Breakdown Brief's Non-Goals: there is no in-product decline; a proposal the client is not ready to accept simply stays open until Nadia revises and re-sends it or the project is cancelled (FEAT-25, XBR-25). Request Changes is the product's only structured "not yet" path.
- Editing or voiding the proposal -- owned entirely by Proposal Creation & Sending (FEAT-02); this screen only reads and reacts to the proposal's current state.
- Legally binding e-signature -- deferred per scope-boundaries.md's Deferral note; the timestamped, immutable Accept click is the launch default, and e-signature (FEAT-26) is out of this feature's MVP scope.
- Listing past proposal versions or a change-request history -- the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix confirms a project has at most one active proposal and this feature does not list history; only the current version is ever shown here.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05 (Client Portal Access, portal home) | Owen navigates from the "proposal waiting on him" item on his portal home | The proposal reference for his company's active or most recently accepted proposal |
| External -- emailed proposal link (FEAT-02.SPEC-011, Proposal Sent/Resent Email) | Owen opens the proposal-sent email and follows the link | The specific proposal reference from the email; if that version has since been voided, the screen redirects to the current version |
| FEAT-03.SPEC-002 (Request Changes) | Owen taps "Back" or completes sending a change-request note | Confirmation that the note was sent; proposal state re-loaded |
| FEAT-03.SPEC-006 (Acceptance Confirmation Notification) | Owen taps the confirmation email's "View proposal" CTA | The accepted proposal reference; the screen shows "Accepted on {accepted_at_date}" |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | None -- this screen is not part of Nadia's own workspace; she sees the resulting acceptance status on the project view (FEAT-01), not this screen | No | Attempting to open this screen's link is treated as an out-of-scope link: plain explanation and a fresh-link option (XBR-09), never proposal content |
| Owen (Client Primary Contact) | Full screen -- scope, price, and current status of his own company's proposal (Own-only) | Accept the proposal; open Request Changes (FEAT-03.SPEC-002) | -- |
| Priya (Client Reviewer Contact) | No proposal content -- portal home shows only the project's stage label (e.g., "Proposal accepted"), per the Access Matrix (Proposals & Acceptance: None for Reviewers) | No | Attempting to reach this screen's link shows the project's stage label only, with no scope, price, or Accept/Request Changes controls |
| Dana (Support Operator) | Full screen content, read-only, inside a logged support session (FEAT-31) | No actions -- Accept and Request Changes controls are not rendered | If Dana attempts an action outside the support session's read-only bounds, the action is not available on screen; there is no control to attempt it with |
| Unauthenticated | No | No | Redirected to the client portal's magic-link sign-in (FEAT-05); after signing in as a recognized contact, the user lands on this screen if they are Owen, or on the stage-only view if Priya |
| Expired session | No | No | Magic link is single-use and time-limited (XBR-28); an expired link shows a plain explanation and a "request a fresh link" option (FEAT-05); no proposal content is shown in the meantime |

## Layout and Content

**Header:** Project name and client company name at the top, with the proposal's status label ("Sent" or "Accepted on {date}") shown beside it.

**Body:** A single-column, read-only presentation of the proposal, in this order:
- Scope description -- full text, rendered before any control below it activates
- Price -- the proposal's price in the project's set currency
- Payment schedule summary -- a short, plain-language statement of the payment structure (deposit, per-milestone, on completion, or a mix) referenced from the Payment Schedule, shown for Owen's context ahead of deciding
- Two primary controls, grouped below the price: an "Accept" button and a "Request Changes" button, both large, clearly labeled tap targets (Shared UI Patterns: Decision controls)
- Once accepted: the "Accept" and "Request Changes" controls are replaced in place by a single "Accepted on {date}" marker (Shared UI Patterns: Immutable-record marker), and the price and scope remain visible below it, now in a visually settled, non-editable presentation

**Footer:** None -- both decision controls sit in the body, directly below the price.

### Responsive Behavior

- **Compact breakpoint:** Single-column body as described, full width; the Accept and Request Changes controls stack vertically, Accept above Request Changes, each spanning the full content width for an easy mobile tap target.
- **Medium size class and above:** Body content is capped at a consistent platform-wide reading width and horizontally centered; the Accept and Request Changes controls sit side by side rather than stacked, Accept on the left.
- **Payment schedule summary:** Uniform scaling, no structural change across breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Scope description and price | Screen loads | Reads the current Proposal record's `scope_description`, `price`, `currency`, and `payment_schedule_reference` | Content renders fully; Accept and Request Changes remain inactive until render completes | Loading indicator while content renders, then full content appears |
| Accept button | Tap | 1. Checks eligibility via FEAT-03.SPEC-005 (Acceptance & Access Rules). 2. If eligible, triggers FEAT-03.SPEC-003 (Acceptance Recording). | Button shows a brief loading state during the check and write | Success: button and Request Changes are both replaced by the "Accepted on {date}" marker. Failure: see Edge Cases (already accepted, voided, connectivity). |
| Request Changes button | Tap | Navigates to FEAT-03.SPEC-002 (Request Changes) | Screen transitions to the Request Changes form | Standard navigation transition |
| "Accepted on {date}" marker | None -- display-only | None | None | Non-interactive; communicates the permanent record |
| Payment schedule summary | None -- display-only | None | None | Non-interactive; provides context ahead of the decision |

### Accessibility Notes

- **Focus order:** Header status label -> scope description -> price -> payment schedule summary -> Accept button -> Request Changes button (or, once accepted, the "Accepted on {date}" marker in their place).
- **Content-ready announcement:** When the scope and price finish rendering and the Accept and Request Changes controls become active, this transition is announced to assistive technology so a screen-reader user is not left waiting on a silently inert control.
- **Acceptance announcement:** When acceptance succeeds, the "Accepted on {date}" marker replacing the controls is announced as a live-region update.
- **Keyboard alternatives:** Accept and Request Changes are both standard activatable controls reachable and operable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this screen exists only once a proposal has been sent; there is no zero-data variant of this screen to render, since a Proposal reference is required to reach it at all | Never entered | Never entered |
| Loading | Scope, price, and payment schedule summary show a loading indicator; Accept and Request Changes are not yet rendered | Screen first opens | Proposal content finishes loading |
| Ready to decide (default) | Full scope, price, and payment schedule summary shown; Accept and Request Changes are both active | Content load completes and the proposal's status is Sent | Owen taps Accept (success) or navigates to Request Changes |
| Accepted | Scope and price remain visible; Accept and Request Changes are replaced by the "Accepted on {date}" marker | Acceptance is successfully recorded (this session or a prior one) | Never exits -- this is a permanent, terminal state for this proposal version |
| Error (accept failed) | Inline message below the controls: "We couldn't record your acceptance. Check your connection and try again." Controls remain active for retry. | The accept action fails mid-flight | Owen retries successfully, or navigates away |
| Voided -- redirected | Brief message "This proposal has been updated" before the current version loads | Owen opens a proposal version that was edited and re-sent after this link was generated | Redirect completes and the current version's Ready-to-decide (or Accepted) state loads |
| Offline/Degraded | Scope and price remain visible from the last successful load; Accept and Request Changes are visibly disabled with the message "Reconnect to accept" beneath them | Connectivity is lost while this screen is open | Connectivity returns -- controls re-activate automatically, no false success is ever shown |

## Validation Rules

Validation and eligibility governed by FEAT-03.SPEC-005 (Acceptance & Access Rules). See that spec for accept-once enforcement, voided-proposal enforcement, and role-based access rules. This screen calls that eligibility check at the moment Accept is tapped, not before.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Request Changes button tap | FEAT-03.SPEC-002 (Request Changes) | -- |
| Successful acceptance | Stays on this screen, now in the Accepted state | -- |
| Voided-version redirect | This same screen, reloaded against the current proposal version | -- |
| Return navigation from FEAT-03.SPEC-002 | This screen, reloaded | -- |

## Data Model

**Creates:** None.
**Reads:** Proposal -- `scope_description`, `price`, `currency`, `payment_schedule_reference`, `status`, `sent_at`, `accepted_at`, `accepted_by`. Client Contact -- the signed-in contact's `role` and `status`, to confirm the viewer is a Primary contact in Active status.
**Updates:** None directly -- Accept triggers FEAT-03.SPEC-003, which performs the Proposal update.
**Deletes:** None.

## Business Rules

- Eligibility to accept (accept-once, voided-cannot-accept) is governed by FEAT-03.SPEC-005 -- this screen does not duplicate that logic, it calls the check.
- XBR-06: A voided proposal cannot be accepted; this screen redirects Owen to the current version rather than showing the stale one as actionable.
- XBR-08: Only a Primary contact at the owning client may view, accept, or request changes; Priya (Reviewer) never reaches this screen's content.
- XBR-09: Client isolation -- Owen only ever reaches his own company's proposal through this screen; an out-of-scope link shows a plain explanation and a fresh-link option.
- The scope and price render fully before the Accept control activates, preventing an accidental early tap (Feature Breakdown Brief, States).

## Edge Cases

- **Owen taps Accept a second time, or against a version voided in the meantime** -- Reject-with-refresh per FEAT-03.SPEC-005: the screen shows "This proposal has already been accepted" (if already accepted) or redirects to the current version (if voided), without recording a duplicate acceptance. Resolution: reject-with-refresh, per the dependency map's Contention note for the Proposal entity.
- **Two Primary contacts at the same client accept at the same moment** -- Only one acceptance is recorded; the second contact's screen shows "This proposal has already been accepted" instead of a second confirmation, per the dependency map's Proposal Contention note ("two Primary contacts accepting at the same moment yield one acceptance and the second sees 'already accepted'").
- **Connectivity drops immediately after Owen taps Accept** -- The action is retried without recording a duplicate acceptance or losing the click's intent (Feature Breakdown Brief, Side-Effect Inventory); the screen shows the Error state with a retry option rather than a false success.
- **Owen navigates away and returns before deciding** -- The screen re-fetches the proposal's current state; no local draft exists to preserve since this is a read-and-decide screen, not a form.
- **Priya follows a proposal link meant for Owen** -- No proposal content is shown; her portal home shows only the project's stage label, per the Access Matrix.
- **Dana opens this screen inside a support session** -- Full content renders read-only; Accept and Request Changes controls are not rendered at all, so there is no control for Dana to attempt.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-03.SPEC-002 (Request Changes) | Navigation (outbound) | Request Changes button navigates here |
| FEAT-03.SPEC-003 (Acceptance Recording) | Triggers (outbound) | Accept button, once eligible, triggers the immutable acceptance write |
| FEAT-03.SPEC-005 (Acceptance & Access Rules) | References (outbound) | Eligibility, access, and concurrent-action rules for the Accept action |
| FEAT-05 (Client Portal Access) | Navigation (inbound) | Owen arrives from his portal home |
| FEAT-02 (Proposal Creation & Sending) | Navigation (inbound) | Owen arrives from the emailed proposal link; a voided-and-resent proposal redirects here to the current version |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| proposal_viewed_by_client | proposal reference, contact role, time since sent | Screen finishes loading and content is shown to Owen | supports success-metrics.md: "Time to Proposal Acceptance" |
| proposal_accepted | proposal reference, time elapsed since sent_at | FEAT-03.SPEC-003 confirms the acceptance was written | supports success-metrics.md: "Time to Proposal Acceptance" |
| proposal_accept_failed | failure reason (connectivity, already accepted, voided) | The accept action does not complete successfully | N/A -- no connected success metric measures acceptance failures; recorded here as diagnostic-only exhaust from the accept flow |

## Acceptance Criteria

**FEAT-03.SPEC-001-AC-01:** Given Owen opens the proposal link from his portal home, when the screen loads, then the scope description and price render fully before the Accept and Request Changes controls become active.

**FEAT-03.SPEC-001-AC-02:** Given Owen is on this screen with the proposal in Sent status, when he taps Accept, then the acceptance is recorded and the Accept and Request Changes controls are replaced by an "Accepted on {date}" marker.

**FEAT-03.SPEC-001-AC-03:** Given Owen is on this screen, when he taps Request Changes, then he is navigated to FEAT-03.SPEC-002 (Request Changes).

**FEAT-03.SPEC-001-AC-04:** Given Owen is on a proposal already showing "Accepted on {date}", when he views the screen, then no Accept or Request Changes control is shown.

**FEAT-03.SPEC-001-AC-05:** Given Owen taps Accept on a proposal that was already accepted by another Primary contact moments earlier, when the eligibility check runs, then the screen shows "This proposal has already been accepted" and no duplicate acceptance is recorded.

**FEAT-03.SPEC-001-AC-06:** Given Owen opens a proposal link for a version that Nadia has since edited and re-sent, when the screen loads, then he is redirected to the current version of the proposal.

**FEAT-03.SPEC-001-AC-07:** Given Owen taps Accept and loses connectivity mid-flight, when connectivity drops, then the screen shows an inline error message with a retry option and no acceptance is recorded until a retry succeeds.

**FEAT-03.SPEC-001-AC-08:** Given Owen loses connectivity while viewing the screen, when he attempts to tap Accept, then the controls are visibly disabled with the message "Reconnect to accept."

**FEAT-03.SPEC-001-AC-09:** Given Priya (Reviewer) follows a proposal link meant for Owen, when her portal home loads, then she sees only the project's stage label and no scope, price, Accept, or Request Changes control.

**FEAT-03.SPEC-001-AC-10:** Given Dana (Support Operator) opens this screen inside a logged support session, when the screen loads, then the full proposal content renders but no Accept or Request Changes control is shown.

**FEAT-03.SPEC-001-AC-11:** Given an unauthenticated visitor opens the proposal link, when the screen would otherwise load, then they are redirected to the client portal's magic-link sign-in.

**FEAT-03.SPEC-001-AC-12:** Given Owen's sign-in session has expired, when he opens the proposal link, then he sees a plain explanation and a "request a fresh link" option, with no proposal content shown.

**FEAT-03.SPEC-001-AC-13:** Given Owen is on the screen in the Ready-to-decide state, when he views the layout at a compact breakpoint, then the Accept and Request Changes controls are stacked vertically, Accept above Request Changes.

**FEAT-03.SPEC-001-AC-14:** Given Owen successfully accepts the proposal, when the acceptance is recorded, then a proposal_accepted analytics event is emitted with the proposal reference and time elapsed since sent_at.

**FEAT-03.SPEC-001-AC-15:** Given an out-of-scope contact opens a proposal link that does not belong to their own company, when the screen would otherwise load, then they see a plain explanation and a fresh-link option, never another company's proposal data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (empty, loading, ready to decide, accepted, error, voided-redirected, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
