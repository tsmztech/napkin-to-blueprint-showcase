---
document_type: feature-overview
feature_number: FEAT-30
feature_name: Contextual Help & Guidance
feature_slug: contextual-help-guidance
priority_tier: Nice-to-Have
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 5
screen_count: 3
automation_count: 1
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

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
