---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-33.SPEC-001
spec_name: Referral Mark Display
spec_slug: referral-mark-display
parent_feature: FEAT-33
parent_feature_name: Portal Referral Attribution
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Referral Mark Display

## Overview

**Name:** Referral Mark Display
**ID:** FEAT-33.SPEC-001
**Type:** Logic/Rule
**Purpose:** Governs where, how, and under what conditions the small "Made with Clientroom" mark renders on every client-facing portal page (FEAT-05) and email (FEAT-14), so it stays discreet, consistent, and never overrides the freelancer's own branding.
**Parent Feature:** FEAT-33 -- Portal Referral Attribution
**Governed Entity:** Referral Mark (a presentational element rendered on every client-facing surface, not a persisted data record)

## Scope and Non-Goals

**In Scope:**
- Placement, link destination, and visual-weight rules for the mark, authored once and referenced by every FEAT-05 portal page and FEAT-14 email rather than restated per surface (Feature Breakdown Brief, Shared UI Patterns)
- The mark's relationship to the freelancer's own branding (Branding Profile, FEAT-19), so it never overrides the freelancer's logo or brand colour (XBR-31)
- The mark's fixed visibility across every subscription plan and every client-facing surface, and what it must never expose
- Authorization for who sees the mark (every role that reaches a client-facing surface)

**Non-Goals:**
- Persisting a Referral Attribution record -- excluded here because the mark is purely a rendering rule with no data of its own; the record is created by FEAT-33.SPEC-004, from data FEAT-33.SPEC-002 captures when the mark is followed, not by this spec
- Access rules for the Referral Attribution record itself -- excluded per the Brief's Entity-Lifecycle Coverage Matrix, which reserves that concern to FEAT-33.SPEC-005 (Referral Data Access Restriction); this spec governs only the visible mark, never the underlying attribution data
- Whether paid plans may hide the mark in the future -- excluded per BRIEF.md, Open Questions (pricing): the founder's brief defers this to a later pricing decision this spec cannot make; product-features.md, Validation & Limits states plainly the mark "cannot be hidden" in MVP
- A native-app rendering of the mark -- excluded per scope-boundaries.md (SC-06): the product ships no native apps, so the mark renders only on the web surfaces FEAT-05 and FEAT-14 already serve

## Governed Entity

The mark carries no persisted fields of its own -- it is a fixed, platform-authored rendering rule applied identically wherever FEAT-05 and FEAT-14 already render client-facing content. The "fields" below are the rule's configuration dimensions, not a data-store schema, and are addressed the same way a data entity's fields would be.

**Entity:** Referral Mark (presentational element)
**Source:** Feature Breakdown Brief (feature-overview.md), Shared UI Patterns; product-features.md, FEAT-33 entry

| Field | Data Type | Description |
|-------|-----------|-------------|
| placement_region | enum (fixed) | Where the mark sits within a rendered surface -- always the footer region |
| link_destination | derived | Where the mark navigates when followed -- always into FEAT-33.SPEC-002 (Referral Link Capture) |
| copy_text | text (fixed) | The mark's visible label -- a single, consistent phrase ("Made with Clientroom") |
| visual_weight | enum (fixed) | The mark's visual prominence -- always small and discreet, never sized or coloured to compete with the freelancer's own branding |
| visibility_by_plan | boolean (fixed) | Whether the mark shows on a given Subscription Plan tier -- always true in MVP, on every tier |
| surface_scope | enum (fixed) | Which surfaces render the mark -- every FEAT-05 client-facing portal page and every FEAT-14 client-facing email |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-05.SPEC-003 (Portal Home) | Client Portal Access | On every render of the portal home screen -- the mark renders in the footer region on every visit |
| FEAT-05.SPEC-002 (Link Verification Landing) | Client Portal Access | On every render, including the expired/invalid-link variant, since the mark is part of the surface's own footer, not conditional on portal content loading successfully |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Notifications (Email) | At composition time, for every client-facing email this capability sends -- the mark is appended to the footer of the composed message before delivery |
| FEAT-19 (Freelancer Branding, referenced) | Freelancer Branding | Read-only reference point: this spec's placement and visual-weight rules are checked against the freelancer's current Branding Profile (logo, brand colour) at render time, never the reverse |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| placement_region | Must render in the footer region of the surface, never inline with the freelancer's own branded header or body content | Always | On every render | N/A -- this is a fixed platform rule, not user input; there is no failure state, only the fixed placement | Yes (structural, not a user-facing validation) |
| link_destination | Must resolve to FEAT-33.SPEC-002 (Referral Link Capture), never directly into portal content or client/project data | Always | On every render | N/A -- fixed platform rule | Yes |
| copy_text | No validation beyond data type -- the phrase is a single, fixed, platform-authored string with no per-freelancer input | Always | -- | -- | -- |
| visual_weight | Must never match or exceed the visual prominence of the freelancer's own logo or primary brand colour treatment | Always | On every render, evaluated against the current Branding Profile | N/A -- fixed platform rule enforced at render time, not a user-correctable validation | Yes |
| visibility_by_plan | Must be true for every Subscription Plan tier that exists in MVP | Always | On every render | N/A -- fixed platform rule | Yes |
| surface_scope | Must include every FEAT-05 portal page and every FEAT-14 client-facing email with no surface exempted | Always | On every render across both features | N/A -- fixed platform rule | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Mark never overrides branding | visual_weight, placement_region | The mark's visual_weight is evaluated relative to the Branding Profile's logo and brand_colour fields at render time; if the freelancer's brand colour would visually dominate the same footer region, the mark keeps its own fixed neutral treatment rather than adopting or clashing with the brand colour (XBR-31) | N/A -- resolved automatically by the fixed rendering rule, never surfaced as an error to any viewer |
| Mark never substitutes for branding fallback | visual_weight, placement_region | When the freelancer has no Branding Profile set (default neutral branding, per FEAT-19), the mark still renders with its own fixed treatment -- it never expands to fill the space a missing logo would occupy | N/A -- resolved automatically |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the mark on a client-facing portal page or email | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Always -- wherever the surface itself is reachable by that role (per FEAT-05's and FEAT-14's own access rules) | -- |
| View the mark on a plan-hidden basis | -- (no role) | Never -- the mark cannot be hidden on any Subscription Plan tier in MVP (product-features.md, Validation & Limits) | N/A -- there is no control anywhere in the product for any role, including Nadia, to hide the mark; the question does not arise |
| Follow the mark (navigate into FEAT-33.SPEC-002) | Nadia, Owen, Priya, Dana, and any unauthenticated visitor viewing a client-facing surface | Always, wherever the mark renders | -- |
| Customize the mark's copy, placement, or visual weight | -- (no role) | Never -- these are fixed platform-authored values with no per-freelancer configuration surface | N/A -- Freelancer Branding (FEAT-19) offers no control over the mark; only the freelancer's own logo and brand colour are configurable there |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|--------------------|-------------|-----------------------|
| link_destination | Derived at render time from the current surface's owning Freelancer Account -- the link the visitor follows implicitly carries which portal or email it was rendered on, which FEAT-33.SPEC-002 reads at click time | On every render | No |
| visual_weight | Fixed platform default -- a small, neutral treatment set once for the whole product, never derived from the freelancer's own Branding Profile | Always | No |
| visibility_by_plan | Fixed to true for every current Subscription Plan tier | Always | No -- whether a future paid tier may override this is an open pricing question the brief defers, not a decision this spec makes |

## Business Rules

- XBR-31: The freelancer's logo and brand colour apply to every client-facing screen and email; the referral mark sits alongside the branding without overriding it. This spec is XBR-31's authority for the mark's own placement and visual-weight side of that rule; FEAT-19 remains the authority for the Branding Profile itself.
- XBR-32: The referral mark appears on every client-facing page and email on every plan in MVP, never reveals client, project, or freelancer portal data, and attribution is used only in aggregate. This spec is XBR-32's authority for the mark's rendering; FEAT-33.SPEC-004 and FEAT-33.SPEC-005 are its authority for attribution recording and access, respectively.
- The mark's link destination never exposes portal content to whoever follows it (scope-boundaries.md, SC-03) -- it leads only to FEAT-33.SPEC-002 and onward to the public Referral Landing Page (FEAT-33.SPEC-003), never into any client contact's portal session.
- The mark renders identically regardless of whether the surface itself is in a loading, error, or degraded state -- it is static content bound to the surface's footer template, not to the surface's data-fetch outcome.

## Edge Cases

- **Freelancer has no Branding Profile set (default neutral branding)** -- The mark still renders with its own fixed small treatment; it does not expand or change weight to compensate for the absence of a logo.
- **A client-facing email fails to render images (image-blocking email client)** -- The mark is a text link, not an image, so it is unaffected by image blocking and remains visible and followable.
- **A portal page renders in a degraded or offline state (FEAT-05)** -- The mark, being static footer content, still renders; only the surface's own data content is affected by the degraded state.
- **A freelancer previews her own portal while signed in** -- The mark still renders exactly as a client contact would see it; there is no preview-mode suppression, since the mark cannot be hidden on any plan.
- **A future verified custom domain is in use (FEAT-27, Later)** -- The mark's link_destination still resolves correctly; only the surface's own address changes, not the mark's rendering rule.
- **Two client-facing emails to the same recipient render at once (e.g., a deliverable-ready email and an invoice email sent close together)** -- Each email independently carries its own footer mark per this spec's surface_scope rule; there is no shared or deduplicated rendering across separate emails.

## Acceptance Criteria

**FEAT-33.SPEC-001-AC-01:** Given Owen opens the Portal Home screen (FEAT-05.SPEC-003), when the page renders, then the "Made with Clientroom" mark appears in the footer region.

**FEAT-33.SPEC-001-AC-02:** Given Nadia has set a custom logo and brand colour in her Branding Profile (FEAT-19), when any client-facing page or email renders, then the mark keeps its own fixed small treatment and never adopts, resizes to match, or visually competes with her brand colour.

**FEAT-33.SPEC-001-AC-03:** Given Nadia has no Branding Profile set (default neutral branding), when a client-facing page renders, then the mark still renders with its own fixed treatment, unchanged by the absence of a logo.

**FEAT-33.SPEC-001-AC-04:** Given Priya opens a client-facing email composed by FEAT-14.SPEC-001, when the email is delivered, then the mark appears in the email's footer alongside Nadia's own branding.

**FEAT-33.SPEC-001-AC-05:** Given Dana opens a read-only support session (FEAT-31) for a freelancer's account, when she views that freelancer's portal pages, then the mark renders the same way it would for any client contact.

**FEAT-33.SPEC-001-AC-06:** Given any visitor views a client-facing portal page or email, when they follow the mark, then they are taken into FEAT-33.SPEC-002 (Referral Link Capture), never directly into portal content.

**FEAT-33.SPEC-001-AC-07:** Given Nadia is on a Free-tier Subscription Plan, when any client-facing surface renders, then the mark is shown -- there is no plan-based control anywhere that can hide it.

**FEAT-33.SPEC-001-AC-08:** Given Nadia searches Settings and Branding (FEAT-19) for a way to hide or customize the mark, when she reviews the available branding controls, then no such control exists -- only her logo and brand colour are configurable there.

**FEAT-33.SPEC-001-AC-09:** Given Owen opens a client-facing email whose images are blocked by his email client, when the email renders, then the mark, being a text link, remains visible and followable.

**FEAT-33.SPEC-001-AC-10:** Given a client-facing portal page is in an offline or degraded state (per FEAT-05's own States), when the page renders its footer, then the mark still appears, unaffected by the degraded data state.

**FEAT-33.SPEC-001-AC-11:** Given Owen follows the mark from Nadia's portal page, when FEAT-33.SPEC-002 receives the click, then the link never carries or exposes any of Nadia's client or project data -- only the fact that this surface belongs to her Freelancer Account.

**FEAT-33.SPEC-001-AC-12:** Given the same client contact receives two separate client-facing emails close together, when both render, then each carries its own independent footer mark per this spec's surface_scope rule.

**FEAT-33.SPEC-001-AC-13:** Given Nadia previews her own portal home while signed in as the freelancer, when the page renders, then the mark appears exactly as it would to a client contact, with no preview-mode suppression.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
