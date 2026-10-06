---
document_type: spec
spec_type: screen
spec_id: FEAT-14.SPEC-002
spec_name: Opt-Out Link Landing
spec_slug: opt-out-link-landing
parent_feature: FEAT-14
parent_feature_name: Messaging Consent Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Screen Spec: Opt-Out Link Landing

## Overview

**Name:** Opt-Out Link Landing
**ID:** FEAT-14.SPEC-002
**Type:** Screen
**Purpose:** The page a client lands on after tapping the opt-out link included in a text message, confirming that texting has been turned off.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

## Scope and Non-Goals

**In Scope:**
- The landing page shown after a valid opt-out link tap
- Triggering FEAT-14.SPEC-004 (Opt-Out / STOP Processing) to revoke consent
- The error state for an already-used, expired, or foreign opt-out link

**Non-Goals:**
- Processing the STOP text-reply path -- owned entirely by FEAT-14.SPEC-004; this screen exists only for the link-tap path, since a text reply has no screen to land on.
- Offering a re-grant action on this page -- excluded per this feature's Key Capabilities: re-granting happens only from FEAT-14.SPEC-001 (Consent & Preferences) or at a later booking (FEAT-14.SPEC-003), never as an immediate undo on the confirmation page itself, so a client cannot accidentally reverse a deliberate opt-out in the same tap sequence that requested it.
- Showing the client's other preferences (email address, booking details) -- excluded per scope-boundaries.md SC-15: this page confirms one thing, the opt-out, and does not become a general preferences hub; a client who wants more goes to FEAT-14.SPEC-001 via FEAT-06's "my bookings".

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External (a message sent through FEAT-08.SPEC-012, transactional text messaging capability) | Client taps the opt-out link embedded in any text message | The link's own reference to the client-Pro relationship the consent applies to |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | None -- this page is a confirmation, not an interactive form | -- |
| The Pro (Talia) | No | No | The Pro has no entry point to this page; it exists only for the client who received and tapped the specific link |
| Platform Operator (Support) | No | No | Support's View-only access to consent state is delivered through its own troubleshooting view, never this page (scope-boundaries SC-05) |
| Unauthenticated | Yes (via the link itself) | No | The link is the entire authentication for this page -- no sign-in is required or offered; a client who reaches this page without a valid link sees the Invalid Link state described below, never a sign-in prompt |
| Expired session | N/A -- this page has no session concept | N/A | This page is a single-use landing reached directly by a link; it has no ongoing session to expire |

single-role product — no restricted elements (this screen serves only the Client role; the two "No/No" rows above describe roles with no entry point, not a restriction on an entry point they have)

## Layout and Content

**Header:** None -- this page has no navigation chrome; it is a standalone confirmation destination.

**Body:** A single centered block:
- A confirmation heading: "Texting turned off"
- A plain-language body line: "You won't receive any more texts from {pro_display_name}. Your confirmations and reminders go to your email instead."
- No action buttons -- the page is informational only

**Footer:** None.

### Responsive Behavior

- **Compact and above:** Uniform scaling, no structural change -- the confirmation block is short enough to need no restructuring at any size.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Page load (implicit -- no interactive element) | Client's tap on the opt-out link resolves to this page | Triggers FEAT-14.SPEC-004 (Opt-Out / STOP Processing) for the link's referenced client-Pro relationship | Page shows a brief loading state, then the confirmation | Confirmation heading and body appear once the automation completes |

### Accessibility Notes

- **Focus order:** The confirmation heading receives focus on page load so assistive technology announces it immediately, since there is no other interactive content to navigate to.
- **Announcements:** The confirmation heading and body are announced together on load; the Invalid Link state's message is announced the same way when that state renders instead.
- **Keyboard alternatives:** N/A -- this page has no interactive controls.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief loading indicator in place of the confirmation block | Page first opens while FEAT-14.SPEC-004 processes the revoke | Processing completes |
| Confirmed | "Texting turned off" heading with the plain-language body line | FEAT-14.SPEC-004 successfully revokes consent | Terminal -- the client closes the page; there is nothing further to do |
| Invalid Link | "This link has already been used or is no longer valid." with no further action offered | The link is already used, expired, or does not resolve to a genuine client-Pro relationship | Terminal -- the client closes the page |
| Offline/Degraded | "You're offline. Reconnect and tap the link again to confirm." in place of the confirmation block | The page cannot reach the product to process the revoke because the client's device is offline | Connectivity is restored and the client reopens the link -- there is no queued action on this page, since a compliance-sensitive opt-out must never be silently deferred |

## Validation Rules

Not applicable -- this page accepts no user input. The only validity check is on the link itself, governed by FEAT-14.SPEC-004's processing (whether the link is Issued and unexpired, or already used/expired/foreign).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| None | This page has no outbound navigation -- it is a terminal confirmation destination, not a hub | -- |

## Data Model

**Creates:** None directly -- the underlying Messaging Consent update is performed by FEAT-14.SPEC-004.
**Reads:** Messaging Consent -- enough of the link's reference to resolve which client-Pro relationship the opt-out applies to, and Pro Account -- display_name, to render the confirmation's plain-language body.
**Updates:** None directly (delegated to FEAT-14.SPEC-004).
**Deletes:** None.

## Business Rules

- This page never asks the client to confirm the opt-out a second time -- tapping the link is itself the explicit action; no-grace-period revocation (XBR-15) means the effect is immediate and the page reflects an already-completed action, not a pending one.
- The Invalid Link state never reveals which of "already used," "expired," or "foreign booking" applies -- a single generic message is shown regardless of cause, consistent with the product's pattern for other link-based screens (e.g., FEAT-06.SPEC-002's identical discipline for access links) so that no link's validity state is probed by trial.
- FEAT-14.SPEC-004 is the sole owner of the actual consent write this page triggers; this screen never writes Messaging Consent state itself.

## Edge Cases

- **The same opt-out link is tapped twice (e.g., opened on two devices)** -- The first tap's processing by FEAT-14.SPEC-004 revokes consent and completes; the second tap finds the link already resolved to a revoked state and shows the Invalid Link message, since the link's single-use discipline mirrors FEAT-06.SPEC-007's Access Link pattern -- there is no harm in showing "already used" here, because the underlying consent state is already the safe, revoked outcome either way.
- **The client's texting consent was already Revoked before this tap (e.g., a prior STOP reply)** -- The page still shows the Confirmed state ("Texting turned off"), since the client's intent and the actual state agree; FEAT-14.SPEC-004 treats this as a no-op write with the same user-visible outcome.
- **The link resolves to a client-Pro relationship that no longer exists (the Client record was deleted, per FEAT-13/XBR-19)** -- The Invalid Link state is shown; a deleted client's consent record no longer exists to update, so the page does not attempt to process the tap as a live revoke.
- **Client is offline when tapping the link** -- The Offline/Degraded state is shown with no queued write; per XBR-15's no-grace-period discipline, an opt-out must be processed at the moment the client acts, never silently deferred to a later reconnect, so the client is asked to tap the link again once back online rather than assuming success.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Navigation (inbound) | The opt-out link embedded in every product text lands here |
| FEAT-14.SPEC-004 (Opt-Out / STOP Processing) | Triggers (outbound) | Page load triggers the revoke for the link's referenced relationship |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (outbound) | Enforces this rule's textability determination indirectly: the landing outcome reflects whether FEAT-14.SPEC-004's revoke changed the state that FEAT-14.SPEC-007 evaluates |
| FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) | References (outbound) | This screen's generic Invalid Link message mirrors that spec's single-use, no-cause-disclosed pattern for link-based screens |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| opt_out_link_landing_shown | outcome (confirmed / invalid_link / offline) | Page finishes loading | N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature; retained as an operational signal so opt-out volume via the link path is observable |

## Acceptance Criteria

**FEAT-14.SPEC-002-AC-01:** Given Riley taps a valid, unused opt-out link in a text from Talia, when the page loads, then FEAT-14.SPEC-004 processes the revoke and the page shows "Texting turned off" with the email-fallback explanation naming Talia.

**FEAT-14.SPEC-002-AC-02:** Given Riley taps an opt-out link that was already used, when the page loads, then it shows "This link has already been used or is no longer valid." with no further action offered.

**FEAT-14.SPEC-002-AC-03:** Given Riley taps an opt-out link whose client-Pro relationship no longer exists, when the page loads, then it shows the same Invalid Link message as an already-used link, never revealing the specific cause.

**FEAT-14.SPEC-002-AC-04:** Given Riley's texting consent was already Revoked before this tap, when the page processes the tap, then it still shows "Texting turned off" as a no-op confirmation.

**FEAT-14.SPEC-002-AC-05:** Given Riley is offline when she taps the opt-out link, when the page attempts to load, then it shows "You're offline. Reconnect and tap the link again to confirm." with no write attempted.

**FEAT-14.SPEC-002-AC-06:** Given Riley taps the same valid opt-out link twice from two devices, when the second tap is processed after the first has already revoked consent, then the second device shows the Invalid Link message.

**FEAT-14.SPEC-002-AC-07:** Given Riley reaches this page, when she looks for a way to undo the opt-out immediately, then no such control exists on this page.

**FEAT-14.SPEC-002-AC-08:** Given Riley reaches this page, when she looks for navigation elsewhere on the product, then none is offered -- the page is a terminal confirmation destination.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 1 | 1 |
| States | 4 (loading, confirmed, invalid link, offline) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
