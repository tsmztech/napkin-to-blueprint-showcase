---
document_type: spec
spec_type: screen
spec_id: FEAT-33.SPEC-003
spec_name: Referral Landing Page
spec_slug: referral-landing-page
parent_feature: FEAT-33
parent_feature_name: Portal Referral Attribution
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Referral Landing Page

## Overview

**Name:** Referral Landing Page
**ID:** FEAT-33.SPEC-003
**Type:** Screen
**Purpose:** A visitor who followed the "Made with Clientroom" mark without wanting to sign up sees a short public product page, with a one-step way back to the portal if they arrived from one.
**Parent Feature:** FEAT-33 -- Portal Referral Attribution

## Scope and Non-Goals

**In Scope:**
- A short, static public page describing Clientroom, reachable only by following the referral mark
- A "Sign up" call to action that carries the captured referring-portal reference (FEAT-33.SPEC-002) into sign-up (FEAT-20.SPEC-001)
- A "Back to portal" affordance, shown only when the visitor arrived with an active client-contact portal session, returning them to their portal in one step
- Public, unauthenticated reachability -- no sign-in is required to view this page

**Non-Goals:**
- Any client, project, or freelancer portal content -- excluded per scope-boundaries.md (SC-03): this page is a public product page and never a window into portal content, regardless of who is viewing it
- A localized or multi-language version of this page -- excluded per scope-boundaries.md (SC-20): the product is English-only at launch, so this page carries no localization behavior
- A native-app rendering of this page -- excluded per scope-boundaries.md (SC-06): the product ships no native apps, so this page is web-only, reached through the mobile or desktop browser
- Displaying which freelancer's portal referred the visit -- excluded per product-features.md, Access ("Nadia is never told who signed up from her portal"); the captured reference is carried silently into sign-up, never surfaced on this page

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-33.SPEC-002 (Referral Link Capture) | Visitor follows the "Made with Clientroom" mark on a portal page or email | The captured referring-portal reference (present or absent), carried silently for later use if the visitor signs up; the visitor's own active portal-session state (if any), used only to decide whether "Back to portal" is shown |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Unauthenticated visitor | Yes | Yes -- may tap "Sign up"; no "Back to portal" affordance is shown, since no portal session exists | -- |
| Owen (Client Primary Contact, arrived with an active portal session) | Yes | Yes -- may tap "Sign up" (creating a separate, unrelated Freelancer Account of his own, per FEAT-20.SPEC-001's happy path) and may tap "Back to portal," returning him to Portal Home (FEAT-05.SPEC-003) | -- |
| Priya (Client Reviewer Contact, arrived with an active portal session) | Yes | Yes -- identical to Owen's case above | -- |
| Nadia (Freelancer, already signed in) | Yes | Yes -- may tap "Sign up"; doing so proceeds to FEAT-20.SPEC-001, which applies its own already-signed-in redirect rule there. No "Back to portal" affordance is shown, since Nadia is not a client contact with a portal session | -- |
| Dana (Support Operator) | Yes | Yes, technically -- this page has no awareness of the operator identity and applies no special restriction to it; in practice Dana has no reason to reach this page outside a support session, which never involves following a client-facing mark | -- |
| Expired session (a client contact whose portal session had expired before following the mark) | Yes | Yes -- may tap "Sign up" normally; "Back to portal" is shown but leads to FEAT-05's "request a fresh sign-in link" screen (FEAT-05.SPEC-001) rather than directly into the portal, since no live session remains | The expired session itself is explained on FEAT-05's own screen, not here -- this page only routes the visitor there |

## Layout and Content

**Header:** Product wordmark only -- no navigation menu, since this page exists to be reached exactly one way (following the mark) and is not part of the product's own browsable site structure.

**Body:** A single, centered content block:
- A short headline describing what Clientroom is (e.g., "Clientroom helps freelancers run client work in one place")
- Two to three sentences of supporting description, in plain, non-technical language
- "Sign up" button (primary, full width on compact screens), which navigates into FEAT-20.SPEC-001 carrying the captured referring-portal reference
- "Back to portal" link (secondary, shown only when the visitor arrived with an active or recently-active client-contact portal session), positioned directly below the "Sign up" button

**Footer:** A single line with Terms of Service and Privacy Policy links, consistent with the product's baseline public-page footer.

### Responsive Behavior

- **Compact breakpoint:** Single-column, centered content block, full width; "Sign up" button full width; "Back to portal" link centered directly beneath it.
- **Medium size class and above:** Content block remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| "Sign up" button | Tap | Navigate to FEAT-20.SPEC-001 (Sign-Up & Account Creation), carrying the captured referring-portal reference (if present) | Screen closes | Standard navigation transition into the sign-up form |
| "Back to portal" link (client contact with an active session) | Tap | Navigate to FEAT-05.SPEC-003 (Portal Home) | Screen closes | Standard navigation transition directly into the portal |
| "Back to portal" link (client contact with an expired session) | Tap | Navigate to FEAT-05.SPEC-001 (Request Sign-In Link) | Screen closes | Standard navigation transition; the expired-session explanation and fresh-link request appear there |
| "Terms of Service" / "Privacy Policy" links (footer) | Tap | Navigate to the respective public document | Screen closes | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Headline -> description -> "Sign up" button -> "Back to portal" link (when shown) -> footer links.
- **Dynamic content announcements:** This page has no dynamic data-fetching content beyond its static text, so no loading or error announcements apply; the only conditional element is the presence or absence of "Back to portal," which is fixed at render time and does not change while the page is open.
- **Keyboard alternatives:** Every action on this page is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-------------------|------------------|
| Default (with Back to portal) | Full page content, including the "Back to portal" link | Visitor arrived with an active or recently-active client-contact portal session | Visitor taps "Sign up," taps "Back to portal," or navigates away |
| Default (without Back to portal) | Full page content, "Back to portal" link omitted | Visitor arrived with no portal session (unauthenticated visitor, or an already-signed-in Nadia) | Visitor taps "Sign up" or navigates away |
| Offline/Degraded | If connectivity is lost after the page has already loaded, the static content remains fully visible and readable; "Sign up" and "Back to portal" are disabled with the inline note "This needs a connection. Try again once you're back online." until connectivity returns | Connectivity is lost while this page is open | Connectivity is restored -- both actions re-enable automatically with no re-fetch needed, since the page's own content is static |

## Validation Rules

Not applicable -- this screen collects no user input; both actions are simple navigations with no form fields to validate.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|---------------------------------------|
| "Sign up" tap | FEAT-20.SPEC-001 (Sign-Up & Account Creation) | FEAT-20 (Onboarding / First-Run Setup) |
| "Back to portal" tap (active session) | FEAT-05.SPEC-003 (Portal Home) | FEAT-05 (Client Portal Access) |
| "Back to portal" tap (expired session) | FEAT-05.SPEC-001 (Request Sign-In Link) | FEAT-05 (Client Portal Access) |

## Data Model

**Creates:** None.
**Reads:** Reads only two pieces of session context, neither of which is a persisted entity field: (1) the referring-portal reference captured by FEAT-33.SPEC-002, read only to pass through into the "Sign up" navigation, never displayed on this page; (2) whether the visitor holds an active or recently-active client-contact portal session (FEAT-05), read only to decide whether "Back to portal" is shown.
**Updates:** None.
**Deletes:** None.

## Business Rules

- This page never displays which freelancer's portal referred the visit, consistent with product-features.md's Access statement that no persona is told who signed up from their portal.
- The captured referring-portal reference passes through this page silently -- it is neither displayed nor altered here; FEAT-33.SPEC-002 captured it and FEAT-20.SPEC-004 later hands it off for recording by FEAT-33.SPEC-004.
- "Back to portal" is shown only when the visitor's own portal session state indicates one exists or recently existed -- it is never shown to an unauthenticated visitor or to an already-signed-in Nadia, since neither is a client contact returning to a portal.
- This page carries no client, project, or freelancer-specific data of any kind (scope-boundaries.md, SC-03) -- its content is identical for every visitor regardless of which portal referred them.

## Edge Cases

- **Visitor arrives directly at this page's address without having followed a mark (e.g., a bookmarked or shared link)** -- The page renders identically to the "without Back to portal" default state; no referring-portal reference exists and none is carried forward if the visitor signs up.
- **Client contact's portal session expires while this page is already open** -- "Back to portal" was rendered based on session state at page load; if the visitor taps it after expiry, FEAT-05's own expired-link handling (FEAT-05.SPEC-001) explains the expiry there rather than this page re-checking session state on every render.
- **Visitor taps "Sign up" twice rapidly** -- The second tap has no additional effect; the first navigation to FEAT-20.SPEC-001 already begins, and this page does not attempt a duplicate submission of any kind, since no form data exists to duplicate.
- **No concurrent-edit conflict applies to this screen** -- This screen creates, reads for display, and updates no shared entity; its only reads are ephemeral session context, so the dependency map's Contention notes do not apply here.
- **Visitor loses connectivity, then returns and taps "Sign up"** -- Handled by the Offline/Degraded state; the action is disabled until connectivity is confirmed restored, then proceeds normally.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-33.SPEC-002 (Referral Link Capture) | Navigation (inbound) | Every follow of the mark routes the visitor here immediately after capture |
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Navigation (outbound) | "Sign up" carries the captured referring-portal reference forward |
| FEAT-05.SPEC-003 (Portal Home) | Navigation (outbound) | "Back to portal" returns an active-session client contact directly to their portal |
| FEAT-05.SPEC-001 (Request Sign-In Link) | Navigation (outbound) | "Back to portal" routes an expired-session client contact to request a fresh link |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|------------------|
| referral_landing_viewed | arrived with an active portal session (yes/no) | This page renders | supports success-metrics.md: "Growth Through Referral" |
| referral_landing_signup_clicked | had a captured referring-portal reference (yes/no) | Visitor taps "Sign up" | supports success-metrics.md: "Growth Through Referral" |
| referral_landing_return_to_portal_clicked | -- | Visitor taps "Back to portal" | N/A -- no Stage 2 metric measures portal-return behavior; this is retained purely to confirm the affordance is used, not as a growth-loop signal |

## Acceptance Criteria

**FEAT-33.SPEC-003-AC-01:** Given Owen follows the mark from Nadia's portal home while his portal session is active, when the Referral Landing Page renders, then he sees the public description, a "Sign up" button, and a "Back to portal" link.

**FEAT-33.SPEC-003-AC-02:** Given an unauthenticated visitor follows the mark from a client-facing email with no portal session of their own, when the page renders, then no "Back to portal" link is shown.

**FEAT-33.SPEC-003-AC-03:** Given Priya is on this page with a captured referring-portal reference, when she taps "Sign up", then she is taken to FEAT-20.SPEC-001 with that reference carried forward silently.

**FEAT-33.SPEC-003-AC-04:** Given Owen is on this page with an active portal session, when he taps "Back to portal", then he is taken directly to Portal Home (FEAT-05.SPEC-003).

**FEAT-33.SPEC-003-AC-05:** Given a client contact's portal session had already expired when they followed the mark, when this page renders and they tap "Back to portal", then they are taken to FEAT-05.SPEC-001 (Request Sign-In Link) rather than directly into the portal.

**FEAT-33.SPEC-003-AC-06:** Given Nadia (already signed in) reaches this page, when it renders, then no "Back to portal" link is shown, since she is not a client contact with a portal session.

**FEAT-33.SPEC-003-AC-07:** Given any visitor is on this page, when they look for any indication of which freelancer's portal referred them, then no such indication is present anywhere on the page.

**FEAT-33.SPEC-003-AC-08:** Given a visitor arrives at this page's address without having followed any mark, when it renders, then it shows the same public content with no "Back to portal" link and no referring-portal reference carried forward.

**FEAT-33.SPEC-003-AC-09:** Given a visitor loses connectivity while this page is open, when they attempt to tap "Sign up" or "Back to portal", then both actions are disabled with the note "This needs a connection. Try again once you're back online." until connectivity is restored.

**FEAT-33.SPEC-003-AC-10:** Given this page renders, when it loads, then `referral_landing_viewed` is emitted noting whether the visitor arrived with an active portal session.

**FEAT-33.SPEC-003-AC-11:** Given a visitor taps "Sign up" twice in rapid succession, when the second tap occurs, then it has no additional effect beyond the first navigation already underway.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 3 (with Back to portal, without Back to portal, offline/degraded) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
