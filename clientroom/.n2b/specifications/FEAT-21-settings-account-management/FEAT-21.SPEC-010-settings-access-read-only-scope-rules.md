---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-21.SPEC-010
spec_name: Settings Access & Read-Only Scope Rules
spec_slug: settings-access-read-only-scope-rules
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Settings Access & Read-Only Scope Rules

## Overview

**Name:** Settings Access & Read-Only Scope Rules
**ID:** FEAT-21.SPEC-010
**Type:** Logic/Rule
**Purpose:** Enforces that only Nadia can edit her own account, that Dana's support session sees profile/preference/business values read-only and never sign-in credentials, and that client contacts have no settings surface at all.
**Parent Feature:** FEAT-21 -- Settings & Account Management
**Governed Entity:** Freelancer Account (whole-entity access scope, across every screen in this feature)

## Scope and Non-Goals

**In Scope:**
- The account-wide, cross-screen access and visibility scope for every role touching the Freelancer Account through this feature
- The screen-by-screen read-only rendering rule applied to Dana's support sessions
- The total exclusion rule applied to client contacts

**Non-Goals:**
- Per-field validation rules -- owned by FEAT-21.SPEC-007 (profile/business fields) and FEAT-21.SPEC-008 (notification preferences); this spec governs *who* may reach and act on a screen, not what makes a field value valid
- Opening, closing, or logging a support session -- owned entirely by Operator Support Access (FEAT-31); this spec only consumes the fact that a logged support session is open and applies this feature's own read-only rendering within it
- Any settings capability for a role beyond the four in the Access Matrix -- excluded per scope-boundaries.md SC-01 and SC-02: the product has no internal-staff seat model and no client-side roles beyond Primary and Reviewer

## Governed Entity

**Entity:** Freelancer Account (whole-entity access scope)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| (whole entity) | -- | This spec governs access to the entity as a whole across FEAT-21.SPEC-001 through FEAT-21.SPEC-004, not any single field; per-field rules are FEAT-21.SPEC-007 and FEAT-21.SPEC-008's domain |
| sign-in email, signed-in devices | text / derived | The specific fields this spec singles out as never visible to Dana under any circumstance (feature-dependency-map.md, Freelancer Account, Data Sensitivity: "sign-in credentials never visible to the operator") |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-001 | Account Profile | On screen entry (renders read-only for Dana, hidden entirely from client contacts) and on every save attempt |
| FEAT-21.SPEC-002 | Notification Preferences | On screen entry and on every toggle attempt |
| FEAT-21.SPEC-003 | Login & Security | On screen entry -- this screen is never rendered inside a support session and never shown to client contacts |
| FEAT-21.SPEC-004 | Business Details & Payment Terms | On screen entry and on every save attempt |

## Field Validation Rules

No field validation rules -- this spec governs access and visibility scope, not field content. See FEAT-21.SPEC-007 (profile/business fields) and FEAT-21.SPEC-008 (notification preferences) for field validation.

## Cross-Field Rules

No cross-field validation rules -- this spec's cross-cutting concern is expressed entirely through the Authorization Rules table below, which spans all four screens rather than individual fields.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View own account (profile, preferences, business details, login & security) | Nadia (Freelancer) | Always, her own account only | -- |
| Edit own account (profile, preferences, business details, login & security) | Nadia (Freelancer) | Always, her own account only | -- |
| View profile, notification preferences, business details | Dana (Support Operator) | Read-only, only inside a logged FEAT-31 support session open on that specific freelancer's account | Outside a logged support session, Dana has no path to any freelancer's Settings at all -- there is no standing access |
| Edit profile, notification preferences, business details | Dana (Support Operator) | Never | No save controls, no destructive actions are rendered on any of these three screens for Dana; a direct attempt to submit a change is refused with "Support sessions are read-only." |
| View sign-in email, signed-in devices, or any Login & Security content | Dana (Support Operator) | Never, under any circumstance | Login & Security (FEAT-21.SPEC-003) is never rendered inside a support session; it does not appear in the Settings navigation shell shown to Dana, and no direct link into it exists from a support session |
| View or edit account closure ("Close account") | Dana (Support Operator) | Never | The "Close account" navigation item is never rendered inside a support session -- account closure is unreachable from a read-only session |
| View or edit any Settings screen or content | Owen (Client Primary Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Owen "None" on Branding, Onboarding & Settings; a direct URL/deep-link attempt shows the client portal's shared out-of-scope explanation defined in FEAT-05.SPEC-002 (Error state, per XBR-09): the heading "This link isn't valid anymore", the line "Sign-in links are single-use and time-limited, and yours has expired, already been used, or was requested again since.", and a single "Send me a new link" button. No Settings content is rendered and the message never says Settings exists or that access was denied |
| View or edit any Settings screen or content | Priya (Client Reviewer Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Priya "None" on Branding, Onboarding & Settings; a direct URL/deep-link attempt shows the identical FEAT-05.SPEC-002 out-of-scope explanation as in Owen's row above (heading "This link isn't valid anymore", the same explanation line, and the "Send me a new link" button) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Dana's Settings visibility scope | Derived from whether a FEAT-31 support session is currently open and logged against the freelancer's account | Evaluated on every screen entry attempt | No -- Dana cannot expand her own scope; it is entirely determined by the support session's state |

## Business Rules

- This spec is the single authoritative home for cross-screen access scope in this feature -- FEAT-21.SPEC-001 through FEAT-21.SPEC-004 each reference this spec rather than defining their own authorization logic (Brief, Shared Validation: "SPEC-001 through SPEC-004 all reference SPEC-010 for what to show, hide, or lock").
- Dana's read-only scope is narrower than "View" on the Access Matrix's own cell for "Subscription & Account Data" in one specific respect: sign-in credentials are excluded entirely, never merely read-only, per the dependency map's explicit Data Sensitivity note for the Freelancer Account entity.
- Every support session Dana opens is itself logged and announced to Nadia by email, per XBR-29 (owned by FEAT-31) -- this spec does not duplicate that logging, it only relies on FEAT-31 to gate when Dana's read-only scope is active at all.
- Client contact exclusion is total and unconditional -- there is no partial, own-only, or view-only tier for Owen or Priya on any Settings screen, unlike their Own-only access elsewhere in the product.

## Edge Cases

- **Dana's support session closes (manually or by inactivity timeout) while she has a Settings screen open** -- Her read-only scope is revoked immediately; the screen redirects her out of the freelancer's account view, consistent with FEAT-31's session lifecycle.
- **Dana attempts to construct a direct link into Login & Security while a support session is open** -- The attempt is refused; FEAT-21.SPEC-003 is never rendered inside a support session regardless of how it is reached, per the "never, under any circumstance" condition above.
- **Owen or Priya follows an old bookmarked Settings URL from before a role change or portal redesign** -- The same "None" denial applies, shown as the FEAT-05.SPEC-002 out-of-scope explanation ("This link isn't valid anymore" with the "Send me a new link" button); no Settings content is ever exposed to a client contact regardless of the path taken to reach it.
- **Two support sessions are somehow opened for the same freelancer account at once (an edge FEAT-31 itself may already prevent)** -- This spec's read-only scope applies identically and independently to each; neither session gains any edit capability, since edit is never granted to Dana under any session state.
- **Nadia opens her own Settings while a Dana support session is also open on her account** -- Nadia's full edit access is entirely unaffected by a concurrent read-only support session; the two views are independent, and Dana's view simply reflects Nadia's most recently saved values on its own refresh.

## Acceptance Criteria

**FEAT-21.SPEC-010-AC-01:** Given Nadia is signed in, when she opens any Settings screen, then she has full view and edit access to her own account.

**FEAT-21.SPEC-010-AC-02:** Given Dana (Support Operator) has no support session open on a freelancer's account, when she attempts to reach that freelancer's Settings, then no path exists -- she has no standing access.

**FEAT-21.SPEC-010-AC-03:** Given Dana (Support Operator) is inside a logged FEAT-31 support session on a freelancer's account, when she views Account Profile, Notification Preferences, or Business Details & Payment Terms, then she sees the current values with no save controls and no destructive actions.

**FEAT-21.SPEC-010-AC-04:** Given Dana (Support Operator) is inside a logged support session, when she attempts to submit a change on any of those three screens through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-010-AC-05:** Given Dana (Support Operator) is inside a logged support session, when she looks at the Settings navigation shell, then "Login & Security" does not appear and no direct link reaches it.

**FEAT-21.SPEC-010-AC-06:** Given Dana (Support Operator) is inside a logged support session, when she looks at the Settings navigation shell, then "Close account" does not appear.

**FEAT-21.SPEC-010-AC-07:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation or follows a direct link to one, then no Settings entry is shown in navigation, and the direct link shows the heading "This link isn't valid anymore" with the line "Sign-in links are single-use and time-limited, and yours has expired, already been used, or was requested again since." and a "Send me a new link" button (FEAT-05.SPEC-002 Error state), with no Settings content exposed.

**FEAT-21.SPEC-010-AC-08:** Given Priya (Client Reviewer Contact) is signed in, when she looks for a Settings entry in navigation or follows a direct link to one, then no Settings entry is shown in navigation, and the direct link shows the identical "This link isn't valid anymore" explanation and "Send me a new link" button, with no Settings content exposed.

**FEAT-21.SPEC-010-AC-09:** Given Dana's support session closes while she has a Settings screen open, when the session ends, then she is redirected out of the freelancer's account view.

**FEAT-21.SPEC-010-AC-10:** Given Nadia has her own Settings open while Dana has a concurrent read-only support session open on the same account, when Nadia saves a change, then Dana's view reflects the new value only on its own next refresh, with no interference to Nadia's save.

**FEAT-21.SPEC-010-AC-11:** Given Dana attempts to construct a direct link into Login & Security while her support session is open, when the link is followed, then the attempt is refused and the screen is not rendered.

**FEAT-21.SPEC-010-AC-12:** Given a client contact follows an old bookmarked Settings URL, when the link is followed, then the same denial applies regardless of the path taken: the heading "This link isn't valid anymore" with the FEAT-05.SPEC-002 explanation line and "Send me a new link" button is shown, and no Settings content is exposed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- owned by FEAT-21.SPEC-007 / FEAT-21.SPEC-008) | 0 |
| Cross-Field Rules | 0 (N/A -- expressed through Authorization Rules) | 0 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
