# FEAT-30 — Contextual Help & Guidance

This chapter covers Contextual Help & Guidance, a Nice-to-Have-tier feature. It contains the feature breakdown brief followed by every specification in full: 5 specifications carrying 71 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-30.SPEC-001 | Contextual Help Tooltip | screen | 15 |
| FEAT-30.SPEC-002 | Freelancer Help Reference | screen | 11 |
| FEAT-30.SPEC-003 | Client Portal Help Reference | screen | 12 |
| FEAT-30.SPEC-004 | Help Tip Dismissal Recording | automation | 11 |
| FEAT-30.SPEC-005 | Contextual Help Content & Behavior Rules | logic-rule | 22 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Contextual Help & Guidance

## Summary

**Feature:** Contextual Help & Guidance
**ID:** FEAT-30
**Description:** Light contextual tooltips and a short help reference for both the freelancer's dashboard and the client-facing portal, so first-time users of either side need no external documentation.
**Priority:** Nice-to-Have
**Phase:** Later
**Type:** Lifecycle
**Rationale:** The decomposition checklist's Commonly Forgotten Areas expects a decided answer on how users learn the product. Nice-to-Have because both sides' flows are designed to be self-explanatory in the moment (one-click accept, one-click approve); phased Later since it is a polish layer added once the core flows are proven rather than a launch requirement. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- On-demand explanation -- a brief inline explanation for an unfamiliar control
- Permanent dismissal -- an experienced user dismisses guidance and it does not reappear

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-30.SPEC-001 | Contextual Help Tooltip | Screen | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | An inline, on-demand explanation for an unfamiliar control, overlaid on dashboard and portal screens at first-encounter moments |
| FEAT-30.SPEC-002 | Freelancer Help Reference | Screen | Nadia (Freelancer) | A short, browsable help reference covering the freelancer dashboard, reachable from any dashboard screen |
| FEAT-30.SPEC-003 | Client Portal Help Reference | Screen | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | A short, browsable help reference covering the client-facing portal, reachable from any portal screen |
| FEAT-30.SPEC-004 | Help Tip Dismissal Recording | Automation | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Persists a user's permanent dismissal of a tip to their Freelancer Account or Client Contact record so it never reappears |
| FEAT-30.SPEC-005 | Contextual Help Content & Behavior Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Governs which guidance content each role may see, that guidance is always advisory (never blocking), and that a dismissed tip is suppressed on every future render |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| On-demand explanation | FEAT-30.SPEC-001 | The tooltip screen renders a brief inline explanation on demand for any unfamiliar control, on both the freelancer dashboard and the client portal | Phase 2 (Explicit) |
| Permanent dismissal | FEAT-30.SPEC-004 | The dismissal automation writes a per-tip, per-user flag to the Freelancer Account or Client Contact record; SPEC-005 checks that flag on every subsequent render | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 2-5:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-30.SPEC-002 | Freelancer Help Reference | Phase 2 (Explicit) | The feature's Description names "a short help reference for... the freelancer's dashboard" as a second surface distinct from the inline tooltips, but it is not itself one of the two Key Capability bullets |
| FEAT-30.SPEC-003 | Client Portal Help Reference | Phase 2 (Explicit) | The feature's Description names "a short help reference for... the client-facing portal" -- a separate audience and content set from SPEC-002, since Owen and Priya have different entitlements (XBR-08) |
| FEAT-30.SPEC-005 | Contextual Help Content & Behavior Rules | Phase 5 (Rule Discovery) | The Validation & Limits field ("advisory only... never blocks"), the Access field (guidance scoped to Nadia and client contacts, never Dana), and XBR-08 (Reviewer contacts never see Primary-only actions) together form a rule set shared across all four other specs -- it crosses the "rules shared across multiple screens or automations" threshold for a standalone Logic/Rule spec |

## Entity-Lifecycle Coverage Matrix

FEAT-30 owns no Connected Entity of its own (Connected Entities: N/A -- "a guidance layer over existing screens, not a data-owning entity"). It does, however, manage a distinct piece of per-user state -- the dismissal flag -- that it creates, reads, and updates on top of the Freelancer Account and Client Contact records. That managed state gets its own CRUD matrix below, per the accepted "feature-local state layered on a host entity" pattern (see FEAT-29's "Feed Item Read State"). Freelancer Account and Client Contact themselves are FEAT-30's Referenced Entities, limited to what this feature reads of them.

**Entity: Help-Tip Dismissal State** (feature-local state -- a per-user, per-tip flag held on the Freelancer Account or Client Contact record, not the account/contact itself; FEAT-30 owns the meaning and lifecycle of this flag even though it is physically stored on another feature's entity)

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-30.SPEC-004 | The flag for a given tip is implicitly created, defaulted to "not dismissed," the first time that tip is eligible to render for a user | Not a separate user-initiated create -- a derived side-effect of a tip's first eligible render |
| Read (single) | FEAT-30.SPEC-005 | Checked per tip, per user, immediately before SPEC-001/SPEC-002/SPEC-003 would render that tip, to decide whether to suppress it | -- |
| Read (list) | N/A | No screen lists a user's dismissal history; Key Capabilities name only "permanent dismissal" behavior, never a review-or-restore list (product-features.md, Key Capabilities) -- recorded as an explicit non-goal | -- |
| Update | FEAT-30.SPEC-004 | Flips the flag from "not dismissed" to "dismissed" when the user permanently dismisses that tip | One-directional: Validation & Limits and the Primary Flow describe only a forward dismissal, never an undismiss action |
| Delete/Archive | Cross-feature -- FEAT-18 (Client Contact) removes the flag when it erases a contact's details on request; FEAT-24 (Freelancer Account) removes the flag when the whole account is deleted | Hard delete, no restore, no independent retention: the flag has no lifecycle of its own apart from its host record -- it is deleted only as a consequence of that record's own delete/erasure path (dependency map: Client Contact "Removed by FEAT-18 (access revoked, details erased on request)"; Freelancer Account "Deleted by FEAT-24") | FEAT-30 defines no purge policy of its own; the flag's retention is entirely inherited from its host record |
| State Transition | FEAT-30.SPEC-004 | "Not dismissed" -> "Dismissed" is the only transition; no reverse transition or intermediate state is defined | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Freelancer Account | FEAT-30.SPEC-001, FEAT-30.SPEC-005 | Host record for Nadia's Help-Tip Dismissal State (product-features.md, FEAT-21 Data Notes and Domain Entity Inventory); FEAT-30 reads it only to check the dismissal flag -- it never reads, creates, updates, or deletes any other field or the account itself |
| Client Contact | FEAT-30.SPEC-001, FEAT-30.SPEC-005 | Host record for Owen's or Priya's Help-Tip Dismissal State, and the source of the Primary/Reviewer role SPEC-005 uses for content scoping (feature-dependency-map.md, Client Contact Fields); FEAT-30 reads it only for these two purposes -- it never creates, updates, or deletes the contact itself |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| User taps/clicks a help affordance on an unfamiliar control | Show the brief inline explanation for that control | Inline in triggering screen | FEAT-30.SPEC-001 |
| User dismisses a tip permanently | Write a dismissal flag for that tip to the user's Freelancer Account or Client Contact record | Standalone Automation | FEAT-30.SPEC-004 |
| Any tooltip, reference entry, or reference page is about to render | Evaluate the viewing role's entitlement and the tip's dismissal state before deciding what (if anything) to show | Standalone Logic/Rule | FEAT-30.SPEC-005 |
| A Client Contact's details are erased on request (FEAT-18) | The contact's dismissal flags are removed together with the rest of the erased record | Cross-feature -- FEAT-18 responsibility | FEAT-18 |
| A Freelancer Account is deleted (FEAT-24) | Nadia's dismissal flags are removed together with the rest of the deleted account | Cross-feature -- FEAT-24 responsibility | FEAT-24 |
| Connectivity is lost while a tooltip or reference is open | Already-loaded content remains available; content not yet loaded is simply unavailable until reconnection (no error, no retry prompt) | Inline in triggering screen | FEAT-30.SPEC-001 / FEAT-30.SPEC-002 / FEAT-30.SPEC-003 |

## Shared Context

**Shared Entities:**
- Freelancer Account -- dismissal flag written by SPEC-004, read by SPEC-001 and SPEC-005; the account itself is owned and otherwise managed by FEAT-20 (creation) and FEAT-21 (all other updates).
- Client Contact -- dismissal flag written by SPEC-004, read by SPEC-001 and SPEC-005; the contact itself is owned and otherwise managed by FEAT-18.

**Shared UI Patterns:**
- Help affordance / tooltip pattern -- SPEC-001 uses one consistent on-demand explanation pattern (an icon or hint that reveals a brief explanation, plus a permanent-dismiss action) across every host screen on both the freelancer dashboard and the client portal, per Spec Writers describing SPEC-001.
- Reference-page layout -- SPEC-002 and SPEC-003 share the same short, browsable reference layout and interaction model; only the audience and content set differ (freelancer-side topics vs. client-side topics), per SPEC-005's role-scoping rule.

**Shared Validation:**
- SPEC-005 defines the role-content-scoping rule, the advisory-only (never-blocking) rule, and the dismissal-suppression rule. SPEC-001, SPEC-002, SPEC-003, and SPEC-004 all reference SPEC-005 rather than each re-deriving these rules.

## Internal Dependency Map

```
SPEC-001 (Contextual Help Tooltip) -> [user dismisses a tip] -> SPEC-004 (Help Tip Dismissal Recording)
SPEC-004 (Help Tip Dismissal Recording) -> [dismissal flag stored] -> SPEC-001 (Contextual Help Tooltip) [suppresses that tip on every future render]
SPEC-001 (Contextual Help Tooltip) -> [applies content and behavior rules from] -> SPEC-005 (Contextual Help Content & Behavior Rules)
SPEC-002 (Freelancer Help Reference) -> [applies content and behavior rules from] -> SPEC-005 (Contextual Help Content & Behavior Rules)
SPEC-003 (Client Portal Help Reference) -> [applies content and behavior rules from] -> SPEC-005 (Contextual Help Content & Behavior Rules)
SPEC-002 (Freelancer Help Reference) -> [dismissal state checked via] -> SPEC-005 (Contextual Help Content & Behavior Rules)
SPEC-003 (Client Portal Help Reference) -> [dismissal state checked via] -> SPEC-005 (Contextual Help Content & Behavior Rules)
```

**Default Entry:** SPEC-001 (Contextual Help Tooltip) -- FEAT-30 has no Navigation Connections row of its own (it is an overlay, not a destination); a user's normal entry point into the feature is encountering an unfamiliar control on a host screen, which surfaces SPEC-001. SPEC-002 and SPEC-003 are reached via a help-reference entry point placed on their respective host dashboards (freelancer dashboard, client portal), not via a feature-level navigation path.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-30.SPEC-001 | Outbound | FEAT-20 (Onboarding / First-Run Setup) | Tooltip overlays first-run setup controls | Nadia's first open reaches an unfamiliar onboarding control |
| FEAT-30.SPEC-001 | Outbound | FEAT-08 (Milestone Approval) | Tooltip overlays the Approve control | A client sees "Approve" for the first time |
| FEAT-30.SPEC-001 | Outbound | FEAT-05 (Client Portal Access) | Tooltip overlays first magic-link sign-in and first portal screens | A contact's first sign-in, or first view of portal content |
| FEAT-30.SPEC-002 | Outbound | FEAT-12 (Freelancer Financial Dashboard) | Help-reference entry point placed on the freelancer dashboard | Nadia opens the help reference from the dashboard |
| FEAT-30.SPEC-004 | Outbound | FEAT-21 (Settings & Account Management) | Writes the dismissal flag onto the Freelancer Account record (FEAT-21 Data Notes lists "help-tip dismissals" among the account's captured fields) | Nadia dismisses a tip |
| FEAT-30.SPEC-004 | Outbound | FEAT-18 (Client Contact Management & Roles) | Writes the dismissal flag onto the Client Contact record | Owen or Priya dismisses a tip |
| FEAT-30.SPEC-005 | Inbound | FEAT-18 (Client Contact Management & Roles) | Reads the contact's Primary/Reviewer role to decide which guidance content that contact may see -- e.g. Priya (Reviewer) is never shown guidance explaining the Approve control, since she has no Approve entitlement (XBR-08); this is the covering behavior for the "Client's First Login and First Feedback" failure/recovery variant | Any tooltip or reference render for a client contact, per XBR-08 |
| FEAT-30.SPEC-001 | Outbound | FEAT-08 (Milestone Approval) | Overlays only the controls a given role actually has -- Owen sees the Approve tooltip; Priya, who lacks Approve access, is never shown it, consistent with the portal already showing her role's scope rather than a confusing broken control | Priya encounters a milestone view that omits an action outside her Reviewer scope |

## Non-Functional Notes

**Data volumes / growth:** The only data FEAT-30 introduces is a small, bounded set of dismissal flags (one per contextual tip) per Freelancer Account or Client Contact record; it rides on those records' existing footprint and does not create a separate growth pattern of its own (assumptions-constraints.md carries no volume expectation naming FEAT-30 directly, and the feature's own Data Notes describe only this bounded per-user flag).

**Responsiveness:** Contextual help is static, already-rendered product content with no loading state of its own (States field: "Loading: N/A -- static contextual content"), so it must never add a perceptible delay to the host screen's own responsiveness target -- notably the client-facing pages FEAT-30.SPEC-001 and FEAT-30.SPEC-003 overlay, which must become interactive within roughly 2 seconds on a typical mobile connection (ASMP-21).

**Data sensitivity / privacy:** The tooltip and reference content itself is static product content with no sensitivity (FEAT-30 Data Notes: "Source: static product content, not user or client data"). The one piece of data FEAT-30 does write -- the per-user dismissal flag -- is stored on the Freelancer Account or Client Contact record and inherits that record's classification: personal data of the freelancer or of individuals at client companies, GDPR-class (ASMP-24), never visible to another client company (ASMP-23).

**Compliance flags:** The dismissal flag is removed, not retained, when a Client Contact's details are erased on request (FEAT-18, dependency map: Client Contact "Removed by FEAT-18 -- access revoked, details erased on request") or when a Freelancer Account is deleted (FEAT-24) -- it is ordinary account data, not evidence, so ASMP-20's evidence-retention exception does not apply to it. Help content itself is English only at launch (scope-boundaries.md, SC-20), consistent with the product's English-only launch scope.

## Non-Goals

- **Configurable, freelancer-authored, or multi-language help content** -- Excluded per scope-boundaries.md SC-11 (no configurable workflow/form/automation builder -- help content is fixed product content, not freelancer-authored) and SC-20 (English only at launch; help content follows the same constraint).
- **A general-purpose chat, messaging, or live-support channel reached from a help tip** -- Excluded per scope-boundaries.md SC-15: comments (FEAT-07) and request-changes notes (FEAT-03) already cover the in-product communication need; contextual help is read-only reference content, not a conversation channel.
- **Operator-side dismissal, reset, or editing of a user's guidance state** -- Excluded per scope-boundaries.md SC-04: Dana's support sessions (FEAT-31) are read-only and logged, and the operator never acts as, or on behalf of, a freelancer or client contact; Dana cannot dismiss or reset tips for anyone.
- **A native-app-only or native-gesture-dependent guidance experience** -- Excluded per scope-boundaries.md SC-06: the product ships no native apps, so both SPEC-001 and SPEC-003 must work as ordinary content in a mobile browser on the client-facing side, with no native-only affordance.
- **A guided, multi-step product tour or sequenced walkthrough engine** -- Excluded by adjacency analysis: the feature's own Rationale states both sides' flows are "designed to be self-explanatory in the moment (one-click accept, one-click approve)," and the Key Capabilities name only a single on-demand explanation and a dismissal, never a sequenced or forced tour; a walkthrough engine would contradict that self-explanatory design premise.
- **A help-engagement analytics dashboard or reporting screen** -- Excluded by adjacency analysis: the feature's Signals (`help_tip_shown`, `help_tip_dismissed`) feed the success-metrics program for the three overlaid features (FEAT-05, FEAT-08, FEAT-20) as raw measurement points; FEAT-30's Key Capabilities and Connected Entities (N/A) name no reporting or dashboard capability of its own.



# Screen Spec: Contextual Help Tooltip

## Overview

**Name:** Contextual Help Tooltip
**ID:** FEAT-30.SPEC-001
**Type:** Screen
**Purpose:** An inline, on-demand explanation for an unfamiliar control, overlaid on a host screen starting at the user's first encounter with the control and offered on every later encounter until permanently dismissed, with a permanent-dismiss action.
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance

## Scope and Non-Goals

**In Scope:**
- The help-affordance icon that appears next to an unfamiliar control, starting at the first encounter and on every later encounter until permanent dismissal, on any freelancer-dashboard or client-portal host screen
- The on-demand explanation popover triggered by that affordance
- The permanent-dismiss action, which triggers FEAT-30.SPEC-004
- A temporary close that leaves the tip eligible to reappear on a later encounter
- Content and role-scoping decisions delegated to FEAT-30.SPEC-005

**Non-Goals:**
- A guided, multi-step product tour or sequenced walkthrough engine -- excluded by adjacency analysis: the feature's Rationale states both sides' flows are "designed to be self-explanatory in the moment (one-click accept, one-click approve)"; a forced or sequenced tour would contradict that design premise.
- Reviewing or restoring a previously dismissed tip -- excluded per the Entity-Lifecycle Coverage Matrix's Read (list) row: "No screen lists a user's dismissal history"; the Key Capabilities name only forward, permanent dismissal, never a review-or-restore list.
- Operator-visible or operator-actionable guidance -- excluded per scope-boundaries.md SC-04: Dana's support sessions are read-only and she never acts as, or on behalf of, a freelancer or client contact, so this overlay never renders in her support-session view.
- A general-purpose chat, messaging, or live-support channel reached from the tip -- excluded per scope-boundaries.md SC-15: comments (FEAT-07) and request-changes notes (FEAT-03) already cover in-product communication.

## Entry Points

This is an overlay, not a destination screen -- it has no default entry of its own. It is instantiated on host screens at every encounter of a given control while the tip is eligible (FEAT-30.SPEC-005 Eligibility to render); the rows below name the first-encounter moment for each host.

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-20 (Onboarding / First-Run Setup) -- any first-run setup screen | Nadia's first open of that screen reaches an unfamiliar setup control | The control's tip_id and the current host screen's identity |
| FEAT-08 (Milestone Approval) -- the milestone approval screen | Owen sees the Approve control for the first time | The control's tip_id (Approve control) |
| FEAT-05 (Client Portal Access) -- first magic-link sign-in and first portal screens | A contact's first sign-in, or first view of portal content | The control's tip_id and the current host screen's identity |
| Any other host screen on the freelancer dashboard or client portal | The viewing user's first encounter with a control this feature defines a tip for | The control's tip_id |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full -- every tip scoped to a control her role can use | Open, temporarily close, and permanently dismiss any tip she is shown | -- |
| Owen (Client Primary Contact) | Own-only -- tips scoped to the controls his Primary role has access to on his own client's portal | Open, temporarily close, and permanently dismiss any tip he is shown | -- |
| Priya (Client Reviewer Contact) | Own-only -- tips scoped to the controls her Reviewer role has access to; never a tip explaining a Primary-only control (e.g., Approve), because FEAT-30.SPEC-005 excludes it and the host screen itself never renders that control to her (XBR-08) | Open, temporarily close, and permanently dismiss any tip she is shown | -- |
| Dana (Support Operator) | No | No | This overlay never renders in a support session; per the Access Matrix, Dana's Notifications & Help entitlement is limited to delivery warnings, and per scope-boundaries.md SC-04 she never acts as, or on behalf of, a freelancer or client contact. She sees the underlying host screen's data read-only, without the guidance layer. |
| Unauthenticated | No | No | The host screen itself is unreachable while unauthenticated (per that screen's own Access rules); since this overlay only ever renders on top of an already-loaded host screen, an unauthenticated visitor never encounters it. |
| Expired session | No | No | The host screen redirects to its own re-authentication path (e.g., FEAT-05's expired-link page for portal screens) before any overlay could render; no tip state is lost because none was in progress. |

## Layout and Content

**Help affordance:** A small, consistent icon (e.g., an info glyph) positioned immediately adjacent to the unfamiliar control it explains, on the host screen, at every encounter of the control while the tip is eligible (from the first encounter until permanent dismissal). The icon is part of the host screen's own layout region for that control -- it does not add a new layout region of its own.

**Popover (on activation):** An inline callout anchored to the affordance, positioned so it does not obscure the control it explains. Contains, top to bottom:
- A brief explanation (one to three sentences) of what the adjacent control does, in plain language scoped to the viewing role's own entitlements (FEAT-30.SPEC-005)
- Two actions, left-aligned in the popover's footer: "Got it" (temporary close) and "Don't show this again" (permanent dismiss)
- A close (X) control in the popover's top-right corner, equivalent to "Got it"

Only one popover is open at a time per screen; opening a second affordance's popover closes any popover already open.

### Responsive Behavior

- **Compact size class:** The popover anchors below the affordance and spans the available width up to a consistent platform-wide maximum, so it never runs off-screen; it never covers the control it explains.
- **Medium size class and above:** The popover anchors directly beside or below the affordance depending on available space, at a fixed, consistent platform-wide width; no structural change beyond positioning.
- **Affordance icon:** Uniform scaling, no structural change across breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Help affordance icon | Tap/click | Opens the popover for this control's tip, per FEAT-30.SPEC-005's content and eligibility rules | Popover appears anchored to the affordance | Explanation text visible in the popover |
| Help affordance icon (popover already open for this tip) | Tap/click | Closes the popover (toggle) | Popover disappears | Host screen returns to its unmodified view |
| "Got it" / close (X) | Tap/click | Closes the popover without recording a dismissal | Popover disappears; nothing is recorded, and the affordance is offered again at the next encounter and every later one until permanent dismissal | Host screen returns to its unmodified view |
| Tap/click outside the popover | Tap/click | Closes the popover, same as "Got it" | Popover disappears | Host screen returns to its unmodified view |
| "Don't show this again" | Tap/click | Closes the popover and triggers FEAT-30.SPEC-004 to permanently record the dismissal for this tip_id and this user | Popover disappears; affordance is removed for this tip on every future render for this user | Host screen returns to its unmodified view; no confirmation message beyond the popover closing (advisory, non-blocking, per FEAT-30.SPEC-005) |

### Accessibility Notes

- **Focus order:** Help affordance icon receives focus in the host screen's existing tab order (immediately after the control it explains); once the popover opens, focus moves into the popover ("Got it" action first, then "Don't show this again," then close); closing the popover by any method returns focus to the affordance icon.
- **Announcements:** Opening the popover announces its explanation text to assistive technology; closing it (by any of the three close paths) announces that the popover has closed and guidance for this control remains available via the affordance.
- **Keyboard alternatives:** The affordance is reachable and activatable by keyboard; the popover's actions are reachable by keyboard; "Escape" closes the popover the same as "Got it." There are no pointer-only gestures on this overlay.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Available (default) | Affordance icon visible next to its control; popover closed | Any encounter (first or later) of the tip's control by a user for whom the tip is eligible per FEAT-30.SPEC-005 (entitled role, catalog entry, not dismissed); this includes every encounter after a temporary close | User taps the affordance, or permanently dismisses the tip |
| Open | Popover visible with explanation text and both close actions | User taps the affordance | User taps "Got it," the close (X), "Don't show this again," or outside the popover |
| Dismissed | Affordance icon is not rendered at all for this tip_id, for this user, on any host screen | FEAT-30.SPEC-004 records a permanent dismissal for this tip_id and user | Never -- dismissal is one-directional (Entity-Lifecycle Coverage Matrix, State Transition row) |
| Loading | N/A -- static contextual content already part of the host screen's own render; there is no separate load step for the tip's explanation text | -- | -- |
| Offline/Degraded | Already-loaded tip content (affordance and, if open, its popover) remains available and interactive; a tip not yet loaded when connectivity was lost is simply not offered on this render -- no error, no retry prompt. A "Don't show this again" chosen while offline closes the popover, suppresses the tip on this device, and is held and retried automatically on reconnection (FEAT-30.SPEC-004) | Connectivity lost while the host screen is open | Connectivity restored -- any held dismissal is retried and the next render re-evaluates tip eligibility normally |

## Validation Rules

Content selection, role scoping, and dismissal-suppression rules are governed by FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules). See that spec for the complete rule set. This screen has no user-entered field input to validate.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Any close action ("Got it," close X, outside tap, or "Don't show this again") | No navigation -- the popover closes and the user remains on the host screen already in view | -- |

## Data Model

**Creates:** None directly on this screen. A Help-Tip Dismissal State record defaulted to "not dismissed" is created as a side effect of a tip's first eligible render -- see FEAT-30.SPEC-004.
**Reads:** Help-Tip Dismissal State (feature-local state on Freelancer Account or Client Contact) -- checked via FEAT-30.SPEC-005 immediately before deciding whether to render a given tip's affordance. Also reads the viewing role from Freelancer Account (Nadia) or Client Contact (Owen's or Priya's Primary/Reviewer role) for content scoping.
**Updates:** None directly -- the "Don't show this again" action triggers FEAT-30.SPEC-004, which owns the write.
**Deletes:** None.

## Business Rules

- Guidance is always advisory, never blocking -- the control this tip explains remains fully usable whether or not its tip has been opened or dismissed (FEAT-30.SPEC-005).
- Content and eligibility for every tip are scoped by role per FEAT-30.SPEC-005 and XBR-08 -- a role never sees a tip for a control it has no entitlement to use.
- Permanent dismissal is one-directional: once "Don't show this again" is chosen, that tip never renders again for that user, on any host screen, on either the freelancer dashboard or the client portal (Entity-Lifecycle Coverage Matrix).
- The affordance is offered at every encounter of its control, from the user's first encounter until permanent dismissal, whenever the tip is eligible to render as defined by FEAT-30.SPEC-005 (entitled role, catalog entry, not dismissed). A temporary close ("Got it," close (X), outside tap, Escape) never ends eligibility; only "Don't show this again" does.

## Edge Cases

- **User taps the affordance twice in rapid succession** -- The second tap toggles the popover closed (per the Interactions table); no duplicate popover opens.
- **Connectivity lost while the popover is open** -- The already-rendered explanation text remains visible and interactive (per the Offline/Degraded state); no error is shown.
- **The "Don't show this again" write (FEAT-30.SPEC-004) fails for a reason other than lost connectivity** -- The popover has already closed optimistically; the write is not retried, so the tip's affordance may reappear on a later encounter because the dismissal was never recorded. No error is shown to the user -- this failure is silent and non-blocking, consistent with the advisory-only rule (FEAT-30.SPEC-005).
- **"Don't show this again" is chosen while the device has no connectivity** -- The popover closes and the tip stays suppressed on this device; FEAT-30.SPEC-004 holds the dismissal and retries it automatically on reconnection, the same retry path as FEAT-30.SPEC-002 and FEAT-30.SPEC-003. Only if the held request is lost before reconnection (device storage cleared or session ended) does the tip reappear at a later encounter; no error is shown in either case.
- **The same user dismisses the same tip from two open sessions at effectively the same time** -- Both writes set the record to the same end state (dismissed = true); there is no meaningful conflict to resolve, and no second confirmation or error is shown.
- **A user's role changes mid-session (e.g., a Client Contact is promoted from Reviewer to Primary by FEAT-18)** -- Tips newly eligible under the new role are offered on the next render that includes the now-accessible controls; dismissal states are tracked per tip_id, so the role change does not retroactively dismiss or restore anything.
- **A control this tip would explain is not shown to the viewing role at all** -- No affordance is offered for it; the host screen never renders the control, so there is nothing to attach a tip to (XBR-08).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-004 (Help Tip Dismissal Recording) | Triggers (outbound) | "Don't show this again" triggers the permanent dismissal write |
| FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) | References (inbound) | Supplies tip content, role scoping, the advisory-only rule, and the dismissal-suppression check |
| FEAT-20 (Onboarding / First-Run Setup) | References (inbound) | Host feature whose first-run setup screens embed this overlay |
| FEAT-08 (Milestone Approval) | References (inbound) | Host feature whose Approve control this overlay explains for Owen |
| FEAT-05 (Client Portal Access) | References (inbound) | Host feature whose first sign-in and first portal screens embed this overlay |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| help_tip_shown | tip_id, host_feature (FEAT-20 \| FEAT-08 \| FEAT-05 \| other), role | Popover opens after an affordance tap | When host_feature = FEAT-20: supports success-metrics.md: "First-Session Activation". When host_feature = FEAT-08: supports success-metrics.md: "Milestone Approval Turnaround". When host_feature = FEAT-05: supports success-metrics.md: "Client Portal Login Success". These three are the overlaid features feature-overview.md's Non-Goals name as consuming this signal as a raw measurement input; FEAT-30 defines no metric or dashboard of its own for tip views on any other host_feature (N/A -- no success-metrics.md metric names a general help-tip-viewing behavior). |
| help_tip_closed_temporarily | tip_id, host_feature | User taps "Got it," the close (X), or outside the popover | N/A -- a temporary close is not itself cited by any success-metrics.md metric; only the shown event (above) and the permanent-dismissal event (FEAT-30.SPEC-004) feed the overlaid features' metrics. |

## Acceptance Criteria

**FEAT-30.SPEC-001-AC-01:** Given Nadia is on a first-run setup screen (FEAT-20) and encounters an unfamiliar control for the first time, when she taps the help affordance beside it, then a popover opens showing a brief explanation of that control.

**FEAT-30.SPEC-001-AC-02:** Given Owen is viewing the milestone approval screen (FEAT-08) for the first time, when he taps the help affordance beside the Approve control, then a popover opens explaining what approving does, and this emits a help_tip_shown event with host_feature FEAT-08.

**FEAT-30.SPEC-001-AC-03:** Given Priya (Reviewer) is on the same milestone view Owen sees, when she looks for a help affordance beside an Approve control, then none is shown, because her role never has an Approve control rendered to it in the first place (XBR-08).

**FEAT-30.SPEC-001-AC-04:** Given Nadia has an open help popover, when she taps "Got it," then the popover closes, no dismissal is recorded, and the affordance is offered again at her next encounter of the control and at every later encounter until she chooses "Don't show this again."

**FEAT-30.SPEC-001-AC-05:** Given Owen has an open help popover, when he taps "Don't show this again," then the popover closes and FEAT-30.SPEC-004 records a permanent dismissal for that tip and Owen.

**FEAT-30.SPEC-001-AC-06:** Given Nadia previously dismissed a tip permanently, when she encounters the same control again on any screen, then no affordance is shown for that tip.

**FEAT-30.SPEC-001-AC-07:** Given Dana is in a read-only support session viewing Nadia's dashboard, when the underlying screen renders, then no help affordances or popovers appear anywhere on it.

**FEAT-30.SPEC-001-AC-08:** Given a visitor is not signed in, when they attempt to reach any host screen this overlay would appear on, then they cannot reach that screen at all, and this overlay never renders.

**FEAT-30.SPEC-001-AC-09:** Given Priya has an open help popover, when she taps outside the popover, then it closes the same as tapping "Got it," and the tip remains available for a later encounter.

**FEAT-30.SPEC-001-AC-10:** Given Owen has one help popover open, when he taps a different control's help affordance, then the first popover closes and the second one opens.

**FEAT-30.SPEC-001-AC-11:** Given Nadia loses connectivity while a help popover is open, when she continues reading it, then the already-rendered explanation stays visible and no error appears.

**FEAT-30.SPEC-001-AC-12:** Given Nadia taps "Don't show this again" and the dismissal write fails for a reason other than lost connectivity, so it is not retried, when she encounters the same control again later, then the affordance may still appear, and no error was ever shown to her for the earlier failed attempt.

**FEAT-30.SPEC-001-AC-13:** Given Priya is promoted from Reviewer to Primary contact by FEAT-18 mid-session, when she next encounters the Approve control, then a help affordance for it is offered to her for the first time, since it is now eligible under her new role.

**FEAT-30.SPEC-001-AC-14:** Given Nadia dismisses the same tip from two open sessions at effectively the same time, when both dismissal writes complete, then the tip is recorded as dismissed exactly once in effect, with no error or conflict shown in either session.

**FEAT-30.SPEC-001-AC-15:** Given Owen taps "Don't show this again" while his device has no connectivity, when the popover closes, then the tip is not offered again on that device, no error is shown, and once connectivity returns FEAT-30.SPEC-004 completes the dismissal automatically so the tip stays suppressed on every future render.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 4 (available, open, dismissed, offline/degraded) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Screen Spec: Freelancer Help Reference

## Overview

**Name:** Freelancer Help Reference
**ID:** FEAT-30.SPEC-002
**Type:** Screen
**Purpose:** A short, browsable help reference covering the freelancer dashboard, reachable from any dashboard screen, so Nadia needs no external documentation.
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance

## Scope and Non-Goals

**In Scope:**
- A browsable list of short reference topics covering the freelancer-side product areas (client and project setup, proposals, milestones and payments, deliverables, invoicing, branding, settings)
- Expand/collapse of each topic to reveal its explanation
- A per-topic "Don't show this again" action for topics that correspond to a contextual tip also shown inline (FEAT-30.SPEC-001), so dismissing it here has the same effect as dismissing it there
- A single, consistent entry point placed on the freelancer dashboard, reachable from every dashboard screen

**Non-Goals:**
- Configurable, freelancer-authored, or multi-language help content -- excluded per scope-boundaries.md SC-11 (help content is fixed product content, not freelancer-authored) and SC-20 (English only at launch).
- A general-purpose chat, messaging, or live-support channel reached from this screen -- excluded per scope-boundaries.md SC-15: comments (FEAT-07) and request-changes notes (FEAT-03) already cover in-product communication.
- A guided, multi-step product tour -- excluded by adjacency analysis: the feature's Rationale states the freelancer flow is "designed to be self-explanatory in the moment," and this reference is a passive lookup, not a forced walkthrough.
- Search across topics -- the reference is deliberately short (product-features.md's Description calls it "a short help reference"); a small, fixed topic set does not warrant a search capability.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Freelancer Financial Dashboard) and every other freelancer dashboard screen | Nadia taps the Help entry point present on every dashboard screen | None -- the reference opens to its default topic list |
| FEAT-12 (Freelancer Financial Dashboard) -- empty dashboard | Zero-state prompt directs a brand-new Nadia toward this reference alongside FEAT-02's proposal prompt | None |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Expand/collapse topics; dismiss any topic that has a corresponding contextual tip | -- |
| Owen (Client Primary Contact) | No | No | Owen operates only within his client portal (FEAT-05) and has no navigation path to the freelancer dashboard; this screen does not exist in his experience. |
| Priya (Client Reviewer Contact) | No | No | Priya operates only within her client portal (FEAT-05) and has no navigation path to the freelancer dashboard; this screen does not exist in her experience. |
| Dana (Support Operator) | No | No | Per the Access Matrix, Dana's Notifications & Help entitlement is limited to delivery warnings; per scope-boundaries.md SC-04 she never acts as, or on behalf of, the freelancer, so this reference is not part of her read-only support-session view. |
| Unauthenticated | No | No | Redirected to Nadia's sign-in screen; after signing in, the user lands on the dashboard (FEAT-12), not this screen directly. |
| Expired session | No | No | Session-expired dialog directs Nadia to sign in again; no in-progress reference state exists to preserve, since this is a static, non-editable screen. |

## Layout and Content

**Header:** Screen title "Help" with a close/back control that returns to the dashboard screen Nadia opened it from.

**Body:** A single-column list of topic entries, grouped under the freelancer-side product areas: Clients and Projects, Proposals, Milestones and Payment Schedules, Deliverables, Invoicing and Payments, Branding, and Account Settings. Each group heading is followed by its topic entries in a fixed, deploy-time order. Each topic entry, collapsed by default, shows only its title; tapping it expands to show its short explanation inline, below the title.

Topics that correspond to a contextual tip also shown inline via FEAT-30.SPEC-001 additionally show, once expanded, a "Don't show this again" action identical in effect to the one on the inline tip. Topics with no corresponding inline tip (broader reference-only content, such as an overview of the invoicing area) show no dismiss action -- they are pure reference content with no dismissal state.

### Responsive Behavior

- **Compact size class:** Single-column list, full width; group headings remain sticky at the top of their group while scrolling.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Help entry point (on host dashboard screens) | Tap | Opens this screen | Screen transitions to the reference list | Reference list appears |
| Close/back control | Tap | Returns to the dashboard screen this was opened from | Screen closes | Animated transition back to the dashboard |
| Topic entry (collapsed) | Tap | Expands the topic to show its explanation | Topic entry shows its explanation inline | Explanation text visible below the topic title |
| Topic entry (expanded) | Tap | Collapses the topic | Explanation hides | Only the title remains visible |
| "Don't show this again" (topics with a corresponding contextual tip only) | Tap | Triggers FEAT-30.SPEC-004 to permanently record the dismissal for that tip_id and Nadia | The corresponding inline tip (FEAT-30.SPEC-001) is suppressed on every future render; the topic stays in the list with its explanation, and its "Don't show this again" action is replaced by the static note "Inline tips for this topic are turned off" (FEAT-30.SPEC-005 Dismissal suppression; no restore control) | No confirmation message beyond the action completing, consistent with the advisory-only rule (FEAT-30.SPEC-005) |

### Accessibility Notes

- **Focus order:** Close/back control -> group headings and topic entries in their displayed order -> (within an expanded topic) "Don't show this again" when present.
- **Announcements:** Expanding or collapsing a topic announces its new state (expanded/collapsed) to assistive technology; a successful dismissal announces that guidance for that topic has been turned off.
- **Keyboard alternatives:** Every topic entry and action on this screen is reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default | Full topic list, all topics collapsed | Screen opens | User expands a topic |
| Topic expanded | The expanded topic shows its explanation (and dismiss action, if applicable); other topics remain collapsed | User taps a collapsed topic | User taps it again, or taps another topic (only one topic is expanded at a time is not required -- multiple topics may be expanded simultaneously) |
| Topic already dismissed | The topic and its full explanation remain in the list and expand/collapse normally; instead of "Don't show this again," the expanded topic shows the static, non-interactive note "Inline tips for this topic are turned off" and no restore control | The topic's tip_id has dismissed = true for Nadia (dismissed earlier here, inline via FEAT-30.SPEC-001, or held offline on this device) | Never -- dismissal is one-directional |
| Loading | N/A -- static contextual content bundled with the screen; there is no separate load step | -- | -- |
| Offline/Degraded | The full topic list and every topic's explanation remain available and interactive, since this is static content already delivered with the screen; only the dismiss action's write may be deferred -- it is held and retried automatically on reconnection per FEAT-30.SPEC-004, and the topic shows as already dismissed on this device meanwhile | Connectivity lost while the screen is open | Connectivity restored -- any deferred dismiss write is retried automatically |

## Validation Rules

This screen has no user-entered field input. The dismiss-eligibility and content-scoping rules governing which topics carry a dismiss action are defined by FEAT-30.SPEC-005.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Close/back control tap | The dashboard screen this was opened from | -- |

## Data Model

**Creates:** None directly. A Help-Tip Dismissal State record defaulted to "not dismissed" is created, when needed, as a side effect of a tip's first eligible evaluation -- see FEAT-30.SPEC-004.
**Reads:** Help-Tip Dismissal State (feature-local state on Freelancer Account) -- checked via FEAT-30.SPEC-005 for each topic that corresponds to a contextual tip, to decide whether to show its "Don't show this again" action (dismissed = false or no record) or the static "Inline tips for this topic are turned off" note (dismissed = true), per FEAT-30.SPEC-005 Dismissal suppression. Reads the fixed help-content catalog for topic titles and explanations.
**Updates:** None directly -- "Don't show this again" triggers FEAT-30.SPEC-004, which owns the write.
**Deletes:** None.

## Business Rules

- Guidance is always advisory, never blocking -- browsing or ignoring this reference never restricts any dashboard capability (FEAT-30.SPEC-005).
- Dismissing a topic here has the identical, one-directional effect as dismissing the same tip inline via FEAT-30.SPEC-001 -- there is exactly one dismissal state per tip_id per user, checked by both surfaces (FEAT-30.SPEC-005).
- Topics with no corresponding inline tip are pure reference content and carry no dismissal state at all -- they always remain in the list.
- Content is fixed, English-only product content (scope-boundaries.md SC-11, SC-20) -- Nadia cannot author, edit, or reorder topics.

## Edge Cases

- **Nadia opens the reference with no connectivity** -- The full topic list and explanations render normally, since this is static content already delivered with the screen; only a dismiss action attempted while offline is held on the device and retried automatically on reconnection (FEAT-30.SPEC-004), with no error shown; if the held request is lost before reconnection, the topic shows its dismiss action again and the inline tip may reappear.
- **Nadia dismisses the same topic from this reference and from its inline tip (FEAT-30.SPEC-001) in quick succession** -- Both writes set the same tip_id to dismissed; the second write is a no-op, and both surfaces show the topic as dismissed with no error.
- **Nadia expands every topic and then collapses them all** -- The list returns to its fully collapsed default with no data loss, since nothing here is user-entered content to preserve.
- **The fixed help-content catalog is updated between one release and the next (new topic added)** -- The new topic appears in the list on the next screen load; no dismissal state exists for it yet, so it renders as not-yet-dismissed by default.
- **Nadia opens this reference before ever encountering a corresponding inline tip** -- The reference entry still shows its dismiss action, since dismissing from the reference is equally valid and suppresses the tip's future inline appearance too.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Contextual Help Tooltip) | References (inbound) | Shares the same underlying tip content and dismissal state for topics that correspond to an inline tip |
| FEAT-30.SPEC-004 (Help Tip Dismissal Recording) | Triggers (outbound) | "Don't show this again" triggers the permanent dismissal write |
| FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) | References (inbound) | Supplies topic content, dismissal-suppression checks, and the advisory-only rule |
| FEAT-12 (Freelancer Financial Dashboard) | Navigation (inbound) | Hosts this screen's entry point, including the empty-dashboard zero-state prompt |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| help_reference_opened | entry_context (dashboard screen name) | Nadia opens this screen | N/A -- no success-metrics.md metric names browsing the freelancer help reference; unlike FEAT-30.SPEC-001's inline tips (which overlay specific measured moments), this is a general lookup with no corresponding Stage 2 metric (feature-overview.md Non-Goals: no help-engagement analytics dashboard is in scope for FEAT-30). |
| help_reference_topic_expanded | topic_id | Nadia expands a topic | N/A -- same reason as help_reference_opened. |

## Acceptance Criteria

**FEAT-30.SPEC-002-AC-01:** Given Nadia is on any dashboard screen, when she taps the Help entry point, then the Freelancer Help Reference opens showing the full topic list, collapsed.

**FEAT-30.SPEC-002-AC-02:** Given Nadia is on the Freelancer Help Reference, when she taps a collapsed topic, then it expands to show its explanation inline.

**FEAT-30.SPEC-002-AC-03:** Given Nadia has expanded a topic that corresponds to an inline tip, when she taps "Don't show this again," then FEAT-30.SPEC-004 records the dismissal and the corresponding inline tip on the dashboard no longer appears on future encounters.

**FEAT-30.SPEC-002-AC-04:** Given Nadia expands a topic with no corresponding inline tip, when she looks for a dismiss action, then none is shown, and the topic always remains in the list.

**FEAT-30.SPEC-002-AC-05:** Given Nadia is on the empty (zero-state) dashboard, when the dashboard renders, then a zero-state prompt directs her toward this reference alongside the prompt to draft a first proposal (FEAT-02).

**FEAT-30.SPEC-002-AC-06:** Given Owen or Priya is signed into their client portal, when they look for any way to reach the Freelancer Help Reference, then no path exists -- this screen is not part of the client portal.

**FEAT-30.SPEC-002-AC-07:** Given Dana is in a read-only support session viewing Nadia's account, when she looks for a help entry point, then none is shown to her.

**FEAT-30.SPEC-002-AC-08:** Given Nadia loses connectivity while this reference is open, when she continues browsing topics, then every topic's explanation remains visible and expandable with no error.

**FEAT-30.SPEC-002-AC-09:** Given Nadia taps "Don't show this again" while offline, when connectivity is lost at that moment, then the dismissal is held on her device, the topic shows the "Inline tips for this topic are turned off" note meanwhile, and FEAT-30.SPEC-004 retries the write automatically once connectivity returns, with no error shown to her.

**FEAT-30.SPEC-002-AC-11:** Given Nadia has already permanently dismissed the tip for a topic (here or inline), when she opens the Freelancer Help Reference and expands that topic, then the topic and its full explanation are shown, the "Don't show this again" action is not rendered, and the static note "Inline tips for this topic are turned off" appears in its place with no restore control.

**FEAT-30.SPEC-002-AC-10:** Given Nadia taps the close/back control, when the screen closes, then she returns to the exact dashboard screen she opened this reference from.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 4 (default, topic expanded, topic already dismissed, offline/degraded) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Client Portal Help Reference

## Overview

**Name:** Client Portal Help Reference
**ID:** FEAT-30.SPEC-003
**Type:** Screen
**Purpose:** A short, browsable help reference covering the client-facing portal, scoped to the viewing contact's role, reachable from any portal screen, so a client contact needs no external documentation.
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance

## Scope and Non-Goals

**In Scope:**
- A browsable list of short reference topics covering the client-portal product areas, scoped to what the viewing contact's role (Primary or Reviewer) can actually do (XBR-08)
- Expand/collapse of each topic to reveal its explanation
- A per-topic "Don't show this again" action for topics that correspond to a contextual tip also shown inline (FEAT-30.SPEC-001)
- A single, consistent entry point placed on the client portal home, reachable from every portal screen

**Non-Goals:**
- Showing Owen and Priya the same topic set -- excluded by design: their entitlements differ (XBR-08), so this reference's content set differs by role, distinct from FEAT-30.SPEC-002's freelancer-only topic set (feature-overview.md Discovery Rationale for this spec).
- A native-app-only or native-gesture-dependent guidance experience -- excluded per scope-boundaries.md SC-06: the product ships no native apps, so this screen must work as ordinary content in a mobile browser.
- Configurable, freelancer-authored, or multi-language help content -- excluded per scope-boundaries.md SC-11 and SC-20: content is fixed, English-only product content.
- A general-purpose chat, messaging, or live-support channel reached from this screen -- excluded per scope-boundaries.md SC-15: comments (FEAT-07) and request-changes notes (FEAT-03) already cover in-product communication.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05 (Client Portal Access) -- portal home, and every other portal screen | The viewing contact taps the Help entry point present on every portal screen | None -- the reference opens to its default topic list, scoped to the viewer's role |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | No | No | Nadia does not sign into a client portal; this screen is not part of her dashboard experience (see FEAT-30.SPEC-002 for her equivalent reference). |
| Owen (Client Primary Contact) | Own-only -- the Primary-scoped topic set for his own client company's portal | Expand/collapse topics; dismiss any topic that has a corresponding contextual tip | -- |
| Priya (Client Reviewer Contact) | Own-only -- the Reviewer-scoped topic set for her own client company's portal, which never includes topics explaining Primary-only actions (accepting, approving, paying, downloading invoices) | Expand/collapse topics; dismiss any topic that has a corresponding contextual tip | -- |
| Dana (Support Operator) | No | No | Per the Access Matrix, Dana's Notifications & Help entitlement is limited to delivery warnings; per scope-boundaries.md SC-04 she never acts as, or on behalf of, a client contact, and she has no client-portal access at all (Client Portal Access row: None). |
| Unauthenticated | No | No | Redirected to the magic-link sign-in request page (FEAT-05); no portal content, including this reference, is reachable before authenticating. |
| Expired session | No | No | The expired/invalid link page (FEAT-05) offers a fresh sign-in link; no in-progress reference state exists to preserve, since this is a static, non-editable screen. |

## Layout and Content

**Header:** Screen title "Help" with a close/back control that returns to the portal screen the contact opened it from.

**Body:** A single-column list of topic entries, grouped under the client-facing product areas relevant to the viewing role. For Owen (Primary), groups include: Proposals and Acceptance, Deliverables and Feedback, Milestone Approval, Invoices and Payments. For Priya (Reviewer), groups include only: Deliverables and Feedback -- her role has no Proposals & Acceptance, Milestone Approval (approve), or Invoicing & Payments entitlement (Access Matrix), so those groups and their topics do not appear in her list at all. Each topic entry, collapsed by default, shows only its title; tapping it expands to show its short explanation inline.

Topics that correspond to a contextual tip also shown inline via FEAT-30.SPEC-001 additionally show, once expanded, a "Don't show this again" action identical in effect to the one on the inline tip. Topics with no corresponding inline tip show no dismiss action.

### Responsive Behavior

- **Compact size class (the default for this mostly-mobile audience):** Single-column list, full width; group headings remain sticky at the top of their group while scrolling.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Help entry point (on host portal screens) | Tap | Opens this screen, scoped to the viewer's role | Screen transitions to the reference list | Reference list appears, showing only the viewer's entitled groups |
| Close/back control | Tap | Returns to the portal screen this was opened from | Screen closes | Animated transition back to the portal screen |
| Topic entry (collapsed) | Tap | Expands the topic to show its explanation | Topic entry shows its explanation inline | Explanation text visible below the topic title |
| Topic entry (expanded) | Tap | Collapses the topic | Explanation hides | Only the title remains visible |
| "Don't show this again" (topics with a corresponding contextual tip only) | Tap | Triggers FEAT-30.SPEC-004 to permanently record the dismissal for that tip_id and the viewing contact | The corresponding inline tip (FEAT-30.SPEC-001) is suppressed on every future render for this contact; the topic stays in the list with its explanation, and its "Don't show this again" action is replaced by the static note "Inline tips for this topic are turned off" (FEAT-30.SPEC-005 Dismissal suppression; no restore control) | No confirmation message beyond the action completing, consistent with the advisory-only rule (FEAT-30.SPEC-005) |

### Accessibility Notes

- **Focus order:** Close/back control -> group headings and topic entries in their displayed order (role-scoped) -> (within an expanded topic) "Don't show this again" when present.
- **Announcements:** Expanding or collapsing a topic announces its new state to assistive technology; a successful dismissal announces that guidance for that topic has been turned off.
- **Keyboard alternatives:** Every topic entry and action on this screen is reachable and activatable by keyboard; there are no pointer-only gestures, consistent with this screen having no native-app-only affordance (SC-06).

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default | Role-scoped topic list, all topics collapsed | Screen opens | User expands a topic |
| Topic expanded | The expanded topic shows its explanation (and dismiss action, if applicable) | User taps a collapsed topic | User taps it again, or taps another topic |
| Topic already dismissed | The topic and its full explanation remain in the role-scoped list and expand/collapse normally; instead of "Don't show this again," the expanded topic shows the static, non-interactive note "Inline tips for this topic are turned off" and no restore control | The topic's tip_id has dismissed = true for the viewing contact (dismissed earlier here, inline via FEAT-30.SPEC-001, or held offline on this device) | Never -- dismissal is one-directional |
| Loading | N/A -- static contextual content bundled with the screen; there is no separate load step | -- | -- |
| Offline/Degraded | The role-scoped topic list and every topic's explanation remain available and interactive, since this is static content already delivered with the screen; only a dismiss action's write may be deferred -- it is held and retried automatically on reconnection per FEAT-30.SPEC-004, and the topic shows as already dismissed on this device meanwhile | Connectivity lost while the screen is open | Connectivity restored -- any deferred dismiss write is retried automatically |

## Validation Rules

This screen has no user-entered field input. The dismiss-eligibility and role-scoping rules governing which topics are shown and which carry a dismiss action are defined by FEAT-30.SPEC-005.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Close/back control tap | The portal screen this was opened from | -- |

## Data Model

**Creates:** None directly. A Help-Tip Dismissal State record defaulted to "not dismissed" is created, when needed, as a side effect of a tip's first eligible evaluation -- see FEAT-30.SPEC-004.
**Reads:** Help-Tip Dismissal State (feature-local state on Client Contact) -- checked via FEAT-30.SPEC-005 for each topic that corresponds to a contextual tip, to decide whether to show its "Don't show this again" action (dismissed = false or no record) or the static "Inline tips for this topic are turned off" note (dismissed = true). Reads the Client Contact's role (Primary or Reviewer) to scope which topic groups appear (XBR-08). Reads the fixed help-content catalog for topic titles and explanations.
**Updates:** None directly -- "Don't show this again" triggers FEAT-30.SPEC-004, which owns the write.
**Deletes:** None.

## Business Rules

- Guidance is always advisory, never blocking -- browsing or ignoring this reference never restricts any portal capability (FEAT-30.SPEC-005).
- Topic groups and topics are scoped to the viewing contact's Primary/Reviewer role exactly as the rest of the portal is scoped (XBR-08, FEAT-30.SPEC-005) -- a role never sees a topic explaining an action it cannot take.
- Dismissing a topic here has the identical, one-directional effect as dismissing the same tip inline via FEAT-30.SPEC-001 -- there is exactly one dismissal state per tip_id per contact.
- Content is fixed, English-only product content (scope-boundaries.md SC-11, SC-20) -- neither Owen nor Priya can author, edit, or reorder topics.
- This screen carries no native-app-only affordance; it works as ordinary content in a mobile browser (scope-boundaries.md SC-06).

## Edge Cases

- **Owen opens the reference with no connectivity** -- The full Primary-scoped topic list renders normally, since it is static content already delivered with the screen; a dismiss action attempted while offline is held on the device and retried automatically on reconnection (FEAT-30.SPEC-004), with no error shown; if the held request is lost before reconnection, the topic shows its dismiss action again and the inline tip may reappear.
- **Priya dismisses the same topic from this reference and from its inline tip (FEAT-30.SPEC-001) in quick succession** -- Both writes set the same tip_id to dismissed for Priya; the second write is a no-op, with no error.
- **A Reviewer contact (Priya) is promoted to Primary by Nadia, who assigns and changes contact roles via FEAT-18 (Owen can only invite Reviewer colleagues, not change roles)** -- On her next open of this screen, the additional Primary-scoped groups (Proposals and Acceptance, Milestone Approval, Invoices and Payments) appear for the first time; her prior dismissals of Reviewer-scoped topics are unaffected.
- **A Primary contact (Owen) invites a new Reviewer colleague (FEAT-18)** -- The new Reviewer's first open of this screen shows only the Reviewer-scoped groups; no dismissal state exists for them yet, so every topic renders as not-yet-dismissed by default.
- **The fixed help-content catalog is updated between one release and the next (new topic added to a role's group)** -- The new topic appears in that role's list on the next screen load, rendering as not-yet-dismissed by default.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Contextual Help Tooltip) | References (inbound) | Shares the same underlying tip content and dismissal state for topics that correspond to an inline tip |
| FEAT-30.SPEC-004 (Help Tip Dismissal Recording) | Triggers (outbound) | "Don't show this again" triggers the permanent dismissal write |
| FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) | References (inbound) | Supplies topic content, role scoping, dismissal-suppression checks, and the advisory-only rule |
| FEAT-05 (Client Portal Access) | Navigation (inbound) | Hosts this screen's entry point on the portal home and every other portal screen |
| FEAT-18 (Client Contact Management & Roles) | References (inbound) | Source of the Primary/Reviewer role this screen's content scoping depends on |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| help_reference_opened | role (Primary \| Reviewer) | The viewing contact opens this screen | N/A -- no success-metrics.md metric names browsing the client portal help reference; unlike FEAT-30.SPEC-001's inline tips (which overlay specific measured moments such as first sign-in), this is a general lookup with no corresponding Stage 2 metric (feature-overview.md Non-Goals: no help-engagement analytics dashboard is in scope for FEAT-30). |
| help_reference_topic_expanded | topic_id, role | The viewing contact expands a topic | N/A -- same reason as help_reference_opened. |

## Acceptance Criteria

**FEAT-30.SPEC-003-AC-01:** Given Owen is on the portal home, when he taps the Help entry point, then the Client Portal Help Reference opens showing his Primary-scoped topic groups, collapsed.

**FEAT-30.SPEC-003-AC-02:** Given Priya is on a portal screen, when she taps the Help entry point, then the reference opens showing only her Reviewer-scoped groups, with no Proposals & Acceptance, Milestone Approval, or Invoicing & Payments group present.

**FEAT-30.SPEC-003-AC-03:** Given Owen has expanded the topic explaining the Approve control, when he taps "Don't show this again," then FEAT-30.SPEC-004 records the dismissal and the corresponding inline tip no longer appears to him on future encounters.

**FEAT-30.SPEC-003-AC-04:** Given Priya expands a topic with no corresponding inline tip, when she looks for a dismiss action, then none is shown, and the topic always remains in her list.

**FEAT-30.SPEC-003-AC-05:** Given Nadia is signed into her freelancer dashboard, when she looks for any way to reach the Client Portal Help Reference, then no path exists -- she uses FEAT-30.SPEC-002 instead.

**FEAT-30.SPEC-003-AC-06:** Given a visitor is not signed in, when they attempt to reach any client-portal screen, then they are redirected to the magic-link sign-in request page, and this reference is unreachable until they sign in.

**FEAT-30.SPEC-003-AC-07:** Given a contact's magic link has expired, when they try to open the portal, then they see the expired-link page with a fresh-link option, and this reference is unreachable until a new sign-in succeeds.

**FEAT-30.SPEC-003-AC-08:** Given Nadia promotes Priya to Primary contact via FEAT-18, when she next opens this reference, then the Primary-scoped groups appear for her for the first time.

**FEAT-30.SPEC-003-AC-09:** Given Owen loses connectivity while this reference is open, when he continues browsing topics, then every topic's explanation remains visible and expandable with no error.

**FEAT-30.SPEC-003-AC-10:** Given Owen taps "Don't show this again" while offline, when connectivity is lost at that moment, then the dismissal is held on his device, the topic shows the "Inline tips for this topic are turned off" note meanwhile, and FEAT-30.SPEC-004 retries the write automatically once connectivity returns, with no error shown.

**FEAT-30.SPEC-003-AC-11:** Given Priya taps the close/back control, when the screen closes, then she returns to the exact portal screen she opened this reference from.

**FEAT-30.SPEC-003-AC-12:** Given Owen has already permanently dismissed the tip for a topic (here or inline), when he opens the Client Portal Help Reference and expands that topic, then the topic and its full explanation are shown, the "Don't show this again" action is not rendered, and the static note "Inline tips for this topic are turned off" appears in its place with no restore control.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 4 (default, topic expanded, topic already dismissed, offline/degraded) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Help Tip Dismissal Recording

## Overview

**Name:** Help Tip Dismissal Recording
**ID:** FEAT-30.SPEC-004
**Type:** Automation
**Purpose:** Owns every write to a user's Help-Tip Dismissal State: the default "not dismissed" record created the first time a tip becomes eligible to render, and the permanent flip to "dismissed" when the user chooses "Don't show this again."
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance

## Scope and Non-Goals

**In Scope:**
- Creating the default "not dismissed" Help-Tip Dismissal State record the first time a given tip_id is eligible to render for a given user
- Recording a user's permanent dismissal of a tip to their Freelancer Account (Nadia) or Client Contact (Owen, Priya) record
- Idempotent handling of a repeated dismissal request for an already-dismissed tip

**Non-Goals:**
- Undismissing or resetting a tip -- excluded per the Entity-Lifecycle Coverage Matrix's State Transition row: "'Not dismissed' -> 'Dismissed' is the only transition; no reverse transition or intermediate state is defined."
- Operator-initiated dismissal or reset on a user's behalf -- excluded per scope-boundaries.md SC-04: Dana's support sessions are read-only, and FEAT-30's own Non-Goals state the operator "cannot dismiss or reset tips for anyone."
- Purging or retaining the dismissal flag independently of its host record -- excluded per the Entity-Lifecycle Coverage Matrix's Delete/Archive row: the flag has no lifecycle of its own; it is removed only as a consequence of FEAT-18's contact erasure or FEAT-24's account deletion.
- Deciding which content or role sees which tip -- that is FEAT-30.SPEC-005's rule set; this automation only persists the flag once that decision is made.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A tip is evaluated for render and no Help-Tip Dismissal State record yet exists for this tip_id and this user | FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) -- evaluated during FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003's render | Fires only when no record exists yet for this exact tip_id + user pairing | tip_id, the user's reference (Freelancer Account for Nadia, Client Contact for Owen or Priya) |
| User chooses "Don't show this again" | FEAT-30.SPEC-001 (Contextual Help Tooltip), FEAT-30.SPEC-002 (Freelancer Help Reference), or FEAT-30.SPEC-003 (Client Portal Help Reference) | Always, on that action | tip_id, the user's reference |

## Processing Logic

1. Receive the tip_id and the acting user's reference (Freelancer Account or Client Contact) from the triggering spec.
2. Determine the host record type from the reference: Freelancer Account for Nadia, Client Contact for Owen or Priya.
3. **Default-initialization path (first-eligible-render trigger):** Check whether a Help-Tip Dismissal State record already exists for this tip_id + host record. If not, create one with dismissed set to false and no dismissed_at value. If one already exists, take no action (nothing to initialize).
4. **Dismissal path (user-initiated trigger):** Check whether a Help-Tip Dismissal State record already exists for this tip_id + host record. If not, create one first (dismissed = false), then proceed. If dismissed is already true, take no further action (idempotent no-op). Otherwise, set dismissed to true and set dismissed_at to the current time.
5. Persist the record. If the write cannot complete because the user's device has lost connectivity, apply the deferral rule: on the **dismissal path** the dismissal request is held on the user's device and retried automatically each time connectivity is restored, until the write completes or is discarded under an Edge Case below; on the **default-initialization path** nothing is held -- the write is simply skipped, and the record is created the next time the tip is evaluated for render with connectivity. Any other write failure (not caused by lost connectivity) is not retried and follows the Persistence failure outcome.
6. For the dismissal path only, signal completion back to the triggering spec so its next render for this tip_id and user suppresses the affordance.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Default record initialized | First-eligible-render trigger fires with no existing record | Creates a Help-Tip Dismissal State record: dismissed = false | None -- this path is silent by design | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 (the tip renders normally as eligible) |
| Dismissal recorded | Dismissal trigger fires and the record's dismissed field is currently false | Sets dismissed = true, dismissed_at = current time | None beyond the popover or reference entry already having closed in the triggering spec (advisory, non-blocking) | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 (that tip is suppressed on every future render for this user) |
| Dismissal already recorded (duplicate) | Dismissal trigger fires and the record's dismissed field is already true | None -- no-op | None -- indistinguishable to the user from a successful dismissal | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 |
| Dismissal deferred (offline) | Dismissal trigger fires while the user's device has no connectivity | None persisted yet; the request is held on the user's device. On this device the tip stays suppressed while the request is held | None -- silent; the triggering screen has already closed optimistically | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 (suppressed on this device immediately; on other devices once the retry succeeds) |
| Deferred dismissal completed | Connectivity is restored while a deferred dismissal is held | Retried automatically; applies the same logic as the Dismissal recorded and Dismissal already recorded outcomes (sets dismissed = true and dismissed_at to the moment the write is applied, or no-op if already true) | None | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 |
| Persistence failure | A default-initialization or dismissal write fails for a reason other than lost connectivity, or a deferred dismissal is lost before it can be retried (see Edge Cases) | None persists; not retried | None -- silent, non-blocking; the triggering screen's UI has already closed optimistically (FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003) | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 (the tip may still appear as eligible on the next render, since the flag was never set) |

## Data Model

**Reads:** Help-Tip Dismissal State -- the existing record (if any) for the given tip_id and host record, to decide whether a write is needed and which branch to take.
**Creates:** Help-Tip Dismissal State -- a new record (tip_id, host record reference, dismissed = false) the first time a tip is eligible to render for a user with no existing record.
**Updates:** Help-Tip Dismissal State -- flips dismissed from false to true and sets dismissed_at, on the Freelancer Account (Nadia) or Client Contact (Owen, Priya) record. No other field of Freelancer Account or Client Contact is read, created, updated, or deleted by this automation.
**Deletes:** None -- the flag's removal is a consequence of FEAT-18 (Client Contact erasure) or FEAT-24 (account deletion), not of this automation.

## Business Rules

- Dismissal is one-directional: this automation defines no path that sets dismissed back to false (Entity-Lifecycle Coverage Matrix, State Transition row).
- This automation is non-blocking: a persistence failure never prevents the triggering screen from closing its popover or reference entry, and never surfaces an error to the user (FEAT-30.SPEC-005's advisory-only rule).
- The default-initialization path never produces user-visible feedback -- it is a derived side effect of a tip's first eligible render, not a user-initiated action (Entity-Lifecycle Coverage Matrix, Create row).
- Repeated dismissal requests for the same tip_id and user are idempotent -- the end state is always dismissed = true, regardless of how many times the request arrives.
- Offline dismissals are deferred, not lost: a dismissal requested from any surface (FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003) while connectivity is lost is held on the user's device and retried automatically on reconnection, identically for all three surfaces; only a dismissal that cannot be retried (device storage cleared or the user's session ended before reconnection) is lost, and then follows the Persistence failure outcome. The first-eligible-render default-initialization write is never deferred.

## Edge Cases

- **Concurrent trigger firing -- two dismiss requests for the same tip_id and user at effectively the same time (e.g., a double tap on "Don't show this again")** -- Both requests resolve to the same end state (dismissed = true); the second to complete is a no-op per the idempotency rule. Neither request is blocked by the other, and no error surfaces.
- **Trigger fires while a previous run for the same tip_id and user is still in flight** -- The triggering screen has already closed its popover or reference entry optimistically on the first tap, so a second identical trigger for the same tip_id and user is deduplicated by the idempotency check in step 4 and produces no additional effect. A trigger for a different tip_id, or a different user, proceeds independently and is never queued behind this one.
- **The first-eligible-render trigger and a dismissal trigger for the same tip_id and user arrive at effectively the same moment** -- The dismissal path's own existence check (step 4) already tolerates "no record yet" by creating one first, so whichever order the two triggers are processed in, the end state is dismissed = true.
- **A deferred dismissal is held and the same user, on another device, dismisses the same tip first** -- The retry finds dismissed already true and is a no-op per the idempotency rule; the first write's dismissed_at stands.
- **A deferred dismissal is held and the user's device storage is cleared or the user's session ends before connectivity returns** -- The held request is lost; the outcome is Persistence failure, so the tip may reappear at a later encounter, and no error is shown.
- **A deferred dismissal is retried after its tip_id has been retired from the catalog** -- Accepted as a no-op per the stale tip_id rule below.
- **The host record (Freelancer Account or Client Contact) is deleted or erased between the trigger firing and the write completing** -- The write is discarded rather than applied to a record that no longer exists; this produces no error, since the flag's entire lifecycle is inherited from its host record's own delete/erasure path (FEAT-18, FEAT-24).
- **A dismissal request references a tip_id that no longer exists in the current fixed help-content catalog (a stale client cached an old tip_id)** -- The write is accepted as a no-op with no error; a tip_id outside the current catalog is never rendered by FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003 regardless of its dismissal state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Contextual Help Tooltip) | Triggered by (inbound) | "Don't show this again" on an inline tip fires the dismissal path |
| FEAT-30.SPEC-002 (Freelancer Help Reference) | Triggered by (inbound) | "Don't show this again" on a reference entry fires the dismissal path |
| FEAT-30.SPEC-003 (Client Portal Help Reference) | Triggered by (inbound) | "Don't show this again" on a reference entry fires the dismissal path |
| FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) | Triggered by (inbound) / References | The first-eligible-render evaluation performed by this spec fires the default-initialization path; this automation's writes are read back by the same spec's suppression rule |
| FEAT-30.SPEC-001 (Contextual Help Tooltip) | Affects (outbound) | Suppresses that tip's affordance on the triggering user's future renders |
| FEAT-18 (Client Contact Management & Roles) | Affects (outbound) | Writes land on the Client Contact record for Owen or Priya |
| FEAT-21 (Settings & Account Management) | Affects (outbound) | Writes land on the Freelancer Account record for Nadia |

## Analytics and Success Signals

- **help_tip_dismissed** (tip_id, host_feature the tip was encountered on (FEAT-20 \| FEAT-08 \| FEAT-05 \| other), role) -- When host_feature = FEAT-20: supports success-metrics.md: "First-Session Activation". When host_feature = FEAT-08: supports success-metrics.md: "Milestone Approval Turnaround". When host_feature = FEAT-05: supports success-metrics.md: "Client Portal Login Success". These are the overlaid features feature-overview.md's Non-Goals names as consuming this signal as a raw measurement input; for any other host_feature: N/A -- no success-metrics.md metric names a general dismissal behavior.
- **help_tip_dismissal_deferred** (tip_id, surface: SPEC-001 \| SPEC-002 \| SPEC-003) -- N/A -- a deferred write is an operational signal, not one any success-metrics.md metric is connected to; it exists only so offline-deferral frequency can be observed internally.
- **help_tip_dismissal_failed** (tip_id, reason: persistence_failure) -- N/A -- a failed write is an operational signal, not one any success-metrics.md metric is connected to; it exists only so the non-blocking guarantee's frequency can be observed internally.

## Acceptance Criteria

**FEAT-30.SPEC-004-AC-01:** Given Nadia encounters a tip for the first time and no Help-Tip Dismissal State record exists yet for it, when FEAT-30.SPEC-005 evaluates it for render, then this automation creates a record with dismissed = false and the tip renders as eligible.

**FEAT-30.SPEC-004-AC-02:** Given a Help-Tip Dismissal State record already exists for a tip and user with dismissed = false, when the same tip is evaluated for render again, then this automation takes no action (no duplicate record is created).

**FEAT-30.SPEC-004-AC-03:** Given Owen taps "Don't show this again" on an inline tip explaining the Approve control (FEAT-08), when this automation processes the dismissal, then it sets dismissed = true and dismissed_at to the current time on Owen's Client Contact record, and emits help_tip_dismissed with host_feature FEAT-08.

**FEAT-30.SPEC-004-AC-04:** Given Priya taps "Don't show this again" on a reference-only topic entry (FEAT-30.SPEC-003) with no corresponding host_feature overlay, when this automation processes the dismissal, then it records the dismissal and emits help_tip_dismissed with the N/A citation, since no overlaid metric applies.

**FEAT-30.SPEC-004-AC-05:** Given a tip is already recorded as dismissed for Nadia, when a second dismissal request for the same tip arrives, then this automation makes no further change and produces no error.

**FEAT-30.SPEC-004-AC-06:** Given Owen taps "Don't show this again" and the write fails for a reason other than lost connectivity, when he next encounters the same control, then the tip may still appear, no retry was made, and no error was shown to him at the time of the failed attempt.

**FEAT-30.SPEC-004-AC-10:** Given Nadia taps "Don't show this again" (from FEAT-30.SPEC-001 or FEAT-30.SPEC-002) while her device has no connectivity, when connectivity is restored, then the held dismissal is retried automatically, dismissed = true and dismissed_at are set, and at no point was an error shown to her; while it was held, the tip stayed suppressed on that device.

**FEAT-30.SPEC-004-AC-11:** Given a tip is evaluated for render while Priya's device has no connectivity and no record exists yet, when the initialization write cannot complete, then nothing is held or retried, and the record is created the next time the tip is evaluated with connectivity.

**FEAT-30.SPEC-004-AC-07:** Given two dismissal requests for the same tip and the same user arrive at effectively the same time, when both are processed, then the end state is dismissed = true exactly once in effect, with no error in either request.

**FEAT-30.SPEC-004-AC-08:** Given a Client Contact's details are erased on request (FEAT-18) after a dismissal write was queued but not yet applied, when the erasure completes first, then the dismissal write is discarded with no error, since the flag's host record no longer exists.

**FEAT-30.SPEC-004-AC-09:** Given a dismissal request references a tip_id no longer present in the current help-content catalog, when this automation processes it, then it is accepted as a no-op and no error is produced.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (first-eligible-render, user dismissal) | 2 |
| Outcome Paths | 6 (default initialized, dismissal recorded, duplicate no-op, dismissal deferred, deferred dismissal completed, persistence failure) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Contextual Help Content & Behavior Rules

## Overview

**Name:** Contextual Help Content & Behavior Rules
**ID:** FEAT-30.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs which guidance content each role may see, that guidance is always advisory and never blocking, and that a dismissed tip is suppressed on every future render.
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance
**Governed Entity:** Help-Tip Dismissal State (feature-local state held on the Freelancer Account or Client Contact record)

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the Help-Tip Dismissal State entity
- The role-content-scoping rule that determines which tips and reference topics a given role may ever be shown
- The advisory-only (never-blocking) rule
- The dismissal-suppression rule
- Authorization rules for every action on the Help-Tip Dismissal State, for every role
- Default values and derivations for the entity's fields

**Non-Goals:**
- The visual layout or interaction pattern of the tooltip or reference screens -- owned by FEAT-30.SPEC-001, FEAT-30.SPEC-002, and FEAT-30.SPEC-003, which reference this spec for the rules they enforce.
- The persistence mechanics of writing a dismissal -- owned by FEAT-30.SPEC-004, which this spec's rules govern but does not itself execute.
- A general-purpose chat, messaging, or live-support channel -- excluded per scope-boundaries.md SC-15, consistent with the rest of FEAT-30.
- Operator-side dismissal, reset, or editing of a user's guidance state -- excluded per scope-boundaries.md SC-04: Dana's support sessions are read-only, and she never acts as, or on behalf of, a freelancer or client contact.

## Governed Entity

**Entity:** Help-Tip Dismissal State
**Source:** Feature Dependency Map (feature-local state layered on Freelancer Account or Client Contact, per FEAT-30's Entity-Lifecycle Coverage Matrix)

| Field | Data Type | Description |
|-------|-----------|-------------|
| tip_id | text | Identifier of the specific contextual tip or reference topic this record's dismissal state applies to; must reference an entry in the fixed, deploy-time help-content catalog |
| host_record | enum (Freelancer Account reference \| Client Contact reference) | The single account or contact this dismissal flag is physically stored on |
| dismissed | boolean | True once the user has permanently dismissed this tip; defaults to false |
| dismissed_at | date/time | The moment dismissed became true; unset while dismissed is false |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-30.SPEC-001 | Contextual Help Tooltip | Immediately before rendering any tip's affordance, and on the "Don't show this again" action |
| FEAT-30.SPEC-002 | Freelancer Help Reference | Immediately before rendering any reference topic's dismiss-eligibility, and on the "Don't show this again" action |
| FEAT-30.SPEC-003 | Client Portal Help Reference | Immediately before rendering the role-scoped topic list and any reference topic's dismiss-eligibility, and on the "Don't show this again" action |
| FEAT-30.SPEC-004 | Help Tip Dismissal Recording | On every write to the Help-Tip Dismissal State (default initialization and the dismissal transition) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tip_id | Must reference an entry in the current fixed help-content catalog | Always | On every read and write | N/A -- an unrecognized tip_id is silently treated as ineligible to render (never shown); see FEAT-30.SPEC-004 Edge Cases | No |
| host_record | Required; must reference exactly one Freelancer Account or exactly one Client Contact, never both | Always | On write (creation) | N/A -- this is a system-set reference, never user-entered, so no user-facing error message applies | No |
| dismissed | Boolean; no validation beyond data type | Always | -- | -- | -- |
| dismissed_at | Must be set if and only if dismissed is true | Always | On write | See Cross-Field Rules below | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Dismissal timestamp consistency | dismissed, dismissed_at | dismissed_at is set exactly when dismissed transitions to true, and remains unset whenever dismissed is false; the two fields can never disagree | N/A -- this is a system-enforced write-time invariant with no user-facing input, so no error message is shown to any user; FEAT-30.SPEC-004 enforces it structurally by only ever setting both together |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read own dismissal state (used internally to decide whether to render a tip or reference-entry dismiss action) | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Always, and only their own record (own tip_id + own host_record) | -- |
| Read dismissal state | Dana (Support Operator) | Never | The contextual help layer is never rendered in a support session; per the Access Matrix, Dana's Notifications & Help entitlement is limited to delivery warnings, so this entity is never read on her behalf. |
| View own dismissal history (a list of previously dismissed tips) | Nadia, Owen, Priya | Never -- no such capability exists | No screen or control exposes a list of past dismissals to any role; the product defines only forward, permanent dismissal, never a review-or-restore list (Entity-Lifecycle Coverage Matrix, Read (list) row). |
| View any user's dismissal history | Dana | Never | No screen or control exposes any user's dismissal history to Dana; her Notifications & Help entitlement is limited to delivery warnings, and this capability does not exist for any role in the first place. |
| Dismiss a tip (set dismissed = true) | Nadia, Owen, Priya | Always, only their own record, and only in the forward direction (not-dismissed -> dismissed) | -- |
| Dismiss a tip on another user's behalf | Nadia, Owen, Priya | Never | No control exists anywhere in the product for one user to dismiss a tip for another; each role sees and dismisses only its own guidance state. |
| Dismiss a tip | Dana | Never | No dismiss control exists in a support session; scope-boundaries.md SC-04 bars the operator from acting as, or on behalf of, a freelancer or client contact. |
| Undismiss or reset a tip (own) | Nadia, Owen, Priya | Never -- no reverse transition is defined | No undismiss or reset control exists anywhere in the product; dismissal is permanent by design (Entity-Lifecycle Coverage Matrix, State Transition row). |
| Undismiss or reset a tip on someone else's behalf | Dana | Never | FEAT-30's Non-Goals explicitly exclude operator-side dismissal, reset, or editing of a user's guidance state. |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| dismissed | Defaults to false | On create -- the first time a given tip_id is eligible to render for a given user (FEAT-30.SPEC-004) | No (the only user-facing transition is the forward dismissal action, which sets it to true; there is no override of the default itself) |
| dismissed_at | Unset by default; set to the current date/time the moment dismissed transitions to true | On the dismissal write only (FEAT-30.SPEC-004) | No |
| tip_id | Direct assignment from the tip or reference topic the user is viewing -- not a default or derived value | On create | No -- it is fixed by which tip triggered the record |
| host_record | Direct assignment from the acting user's own Freelancer Account or Client Contact reference -- not a default or derived value | On create | No -- always the acting user's own record; see Authorization Rules |

## Business Rules

- **Advisory-only, never blocking:** No control's core action is ever gated by whether its tip has been opened or dismissed. The presence, absence, or dismissal state of a tip never changes what a user can do on the host screen.
- **Role-content scoping (XBR-08):** A tip or reference topic is eligible to render for a role only if that role has an entitlement to the control or action it explains. Priya (Reviewer) is never shown a tip or topic explaining a Primary-only action (accept, approve, pay, download invoice, invite a Reviewer) because her role has no entitlement to those actions in the first place -- the host screen never renders the underlying control to her, so no tip attaches to it either.
- **Eligibility to render (the single definition used by FEAT-30.SPEC-001, FEAT-30.SPEC-002, and FEAT-30.SPEC-003):** A tip_id is eligible to render for a user if and only if all three conditions hold: (1) the user's role is entitled to the control or action the tip explains (role-content scoping, XBR-08); (2) the tip_id exists in the current fixed help-content catalog; and (3) the user's Help-Tip Dismissal State for that tip_id has dismissed = false or no record yet. An "encounter" is any render of a host screen region that contains the tip's control. The first encounter of an eligible tip is its first-encounter moment, and the tip's inline affordance is offered at that encounter and at every later encounter -- a temporary close ("Got it," the close (X), or an outside tap) records nothing and never ends eligibility -- until the user permanently dismisses it via "Don't show this again." There is no view count, time limit, or automatic expiry after which an undismissed, entitled tip stops being offered.
- **Dismissal suppression:** Immediately before FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003 would render a given tip_id for a given user, the eligibility rule above is checked. If the Help-Tip Dismissal State for that pairing has dismissed = true, one behavior applies on every surface: (a) the inline affordance (FEAT-30.SPEC-001) is not rendered; (b) on a reference topic (FEAT-30.SPEC-002, FEAT-30.SPEC-003) the "Don't show this again" action is not rendered and is replaced by a static, non-interactive note reading "Inline tips for this topic are turned off," with no restore control; and (c) the reference topic itself stays in the list with its full explanation, always visible and expandable regardless of dismissal state -- dismissal never suppresses the underlying reference content.
- **One-directional lifecycle:** Once dismissed, a tip's state never reverts (Entity-Lifecycle Coverage Matrix, State Transition row); this spec defines no combination of conditions under which dismissed reverts to false.
- **Inherited retention:** This spec defines no purge policy of its own for the Help-Tip Dismissal State; its retention is entirely inherited from its host record's own delete/erasure path (FEAT-18's contact erasure, FEAT-24's account deletion).

## Edge Cases

- **tip_id references a tip retired from the current help-content catalog** -- Treated as ineligible to render regardless of its dismissal state; no error is produced anywhere in the product.
- **The same tip is evaluated twice within a single render (e.g., appears on two regions of a busy screen)** -- The read is idempotent; both evaluations see the same dismissal state and reach the same eligibility decision.
- **A Client Contact's role changes mid-session (Reviewer promoted to Primary, or vice versa demoted, by FEAT-18)** -- Content eligibility re-evaluates against the new role on the next render; dismissal states are tracked per tip_id and are unaffected by the role change -- a newly eligible tip (now accessible under the new role) has no prior dismissal record and renders as not-yet-dismissed by default; a tip that becomes ineligible under a demotion is simply no longer offered, regardless of its dismissal state.
- **A dismissal write is attempted for a tip_id not present in the current catalog (stale cached client)** -- Accepted as a no-op with no error, per FEAT-30.SPEC-004; this spec's eligibility rule would never have rendered that tip_id in the first place.
- **dismissed is true but dismissed_at is somehow missing (a boundary the cross-field rule is designed to prevent)** -- Never occurs under normal operation because FEAT-30.SPEC-004 sets both fields together in the same write; there is no code or user path in this product definition that sets one without the other.
- **Dana's support session views a host screen that, for the account owner, would show a contextual tip** -- The tip's affordance is never rendered in a support session at all, regardless of the account owner's own dismissal state for it, per the Authorization Rules' "Read dismissal state: Dana, Never" row.

## Acceptance Criteria

**FEAT-30.SPEC-005-AC-01:** Given a tip_id exists in the current help-content catalog, when FEAT-30.SPEC-001 checks its record, then the read succeeds and eligibility is decided from the dismissed field.

**FEAT-30.SPEC-005-AC-02:** Given a tip_id has been retired from the current help-content catalog, when any of FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003 evaluate it, then it is treated as ineligible to render, with no error shown anywhere.

**FEAT-30.SPEC-005-AC-03:** Given a Help-Tip Dismissal State record is created for Nadia's own tip, when the host_record is checked, then it references Nadia's Freelancer Account and no other account.

**FEAT-30.SPEC-005-AC-04:** Given a tip has never been dismissed, when its record is checked, then dismissed is false and dismissed_at is unset.

**FEAT-30.SPEC-005-AC-05:** Given a tip has just been dismissed, when its record is checked immediately after, then dismissed is true and dismissed_at holds the moment of dismissal -- the two fields are never found in disagreement.

**FEAT-30.SPEC-005-AC-06:** Given Nadia is viewing her own dismissal state indirectly through FEAT-30.SPEC-001's render check, when the check runs, then it succeeds because she is reading only her own record.

**FEAT-30.SPEC-005-AC-07:** Given Dana is in a support session, when the host screen would otherwise check a dismissal state to render a tip, then that read never occurs and no tip is rendered to her.

**FEAT-30.SPEC-005-AC-08:** Given Nadia, Owen, or Priya looks for a list of tips they have previously dismissed, when they search the product, then no such screen or control exists anywhere.

**FEAT-30.SPEC-005-AC-09:** Given Owen has not yet dismissed a tip, when he taps "Don't show this again," then the transition from not-dismissed to dismissed succeeds.

**FEAT-30.SPEC-005-AC-10:** Given Priya wants to dismiss a tip on Owen's behalf, when she looks for a way to do so, then no such control exists -- each contact dismisses only their own guidance state.

**FEAT-30.SPEC-005-AC-11:** Given Dana wants to dismiss a tip for Nadia during a support session, when she looks for a dismiss control, then none is shown to her, consistent with SC-04.

**FEAT-30.SPEC-005-AC-12:** Given Owen has permanently dismissed a tip, when he looks for a way to undismiss or restore it, then no such control exists anywhere in the product.

**FEAT-30.SPEC-005-AC-13:** Given Dana wants to reset a dismissed tip for Nadia, when she looks for a reset control, then none exists, per FEAT-30's Non-Goals.

**FEAT-30.SPEC-005-AC-14:** Given a tip is eligible to render for Nadia for the first time, when FEAT-30.SPEC-004 initializes its record, then dismissed defaults to false with no user action required.

**FEAT-30.SPEC-005-AC-15:** Given Priya (Reviewer) is viewing a milestone screen, when tip eligibility is evaluated for the Approve control, then no tip is offered to her, because her role has no entitlement to that control at all (XBR-08).

**FEAT-30.SPEC-005-AC-16:** Given a tip has been dismissed by Nadia, when the host screen that would show it renders again, then the tip's affordance is suppressed while the rest of the screen renders normally and unaffected.

**FEAT-30.SPEC-005-AC-17:** Given Owen has dismissed a tip explaining the pay-invoice action, when he next opens the invoice screen, then he can still pay the invoice without restriction -- the dismissal never gates the underlying action, consistent with the advisory-only rule.

**FEAT-30.SPEC-005-AC-18:** Given Priya is promoted from Reviewer to Primary contact (FEAT-18) mid-session, when guidance eligibility is next evaluated for her, then Primary-scoped tips and topics become eligible for the first time, with no retroactive change to her existing dismissal records.

**FEAT-30.SPEC-005-AC-19:** Given a Client Contact's details are erased on request (FEAT-18), when the erasure completes, then their Help-Tip Dismissal State records are removed together with the rest of the erased record, since this spec defines no independent retention for them.

**FEAT-30.SPEC-005-AC-20:** Given Dana is in a support session, when she looks for a way to view Nadia's (or any user's) dismissal history, then no such screen or control exists, consistent with her Notifications & Help entitlement being limited to delivery warnings.

**FEAT-30.SPEC-005-AC-21:** Given Nadia is entitled to a control's tip, the tip_id is in the catalog, and she has tapped "Got it" on it without ever choosing "Don't show this again," when she next encounters the control on any later render, then the affordance is offered again -- and it is offered again on every subsequent encounter until she permanently dismisses it.

**FEAT-30.SPEC-005-AC-22:** Given Owen has permanently dismissed the tip for a topic that appears in the Client Portal Help Reference, when he opens that topic, then the topic and its full explanation are still shown, the "Don't show this again" action is not rendered, and the static note "Inline tips for this topic are turned off" appears in its place with no restore control.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
