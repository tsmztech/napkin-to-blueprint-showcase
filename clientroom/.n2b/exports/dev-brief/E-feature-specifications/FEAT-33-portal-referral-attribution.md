# FEAT-33 — Portal Referral Attribution

This chapter covers Portal Referral Attribution, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 5 specifications carrying 57 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-33.SPEC-001 | Referral Mark Display | logic-rule | 13 |
| FEAT-33.SPEC-002 | Referral Link Capture | automation | 11 |
| FEAT-33.SPEC-003 | Referral Landing Page | screen | 11 |
| FEAT-33.SPEC-004 | Referral Attribution Recording | automation | 12 |
| FEAT-33.SPEC-005 | Referral Data Access Restriction | logic-rule | 10 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Portal Referral Attribution

## Summary

**Feature:** Portal Referral Attribution
**ID:** FEAT-33
**Description:** Every client-facing portal page and email carries a small, discreet "Made with Clientroom" link. A client contact, or a fellow freelancer shown the portal, can follow it to start an account, and every new sign-up records how the freelancer found the product.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** BRIEF.md, Business Context: "The intended growth loop is that new freelancers arrive after a client or peer sees someone else's portal," and Success Criteria: "Most new freelancers arrive because a client or peer saw someone else's portal." No draft feature gave a portal viewer a way to reach the product or measured whether the loop works. Research shows white-labeled branding is highly valued (SuiteDash, HIGH), so the mark stays small and never competes with the freelancer's own brand. Important rather than Core because the client-facing loop works without it; MVP because the brief's growth success criterion must be measurable from the very first portal. [AUDIT-ADDED: 1 -- the brief-goal walk found no feature delivering or measuring the portal-driven growth loop named in BRIEF.md's Success Criteria]

**Key Capabilities:**
- Discreet product mark -- a small "Made with Clientroom" link at the foot of client-facing pages and emails
- Start from a portal -- a visitor who follows the link lands on sign-up, with the referring portal recorded
- Tell us how you heard -- each new freelancer is asked, optionally, how they found Clientroom

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-33.SPEC-001 | Referral Mark Display | Logic/Rule | All | Governs where and how the "Made with Clientroom" mark renders on every client-facing page and email, without overriding the freelancer's own branding |
| FEAT-33.SPEC-002 | Referral Link Capture | Automation | Nadia | Captures the referring freelancer's portal identifier when a visitor follows the mark, and carries it toward sign-up |
| FEAT-33.SPEC-003 | Referral Landing Page | Screen | All | Visitor who follows the mark without wanting to sign up sees a short public product page and can return to the portal in one step |
| FEAT-33.SPEC-004 | Referral Attribution Recording | Automation | Nadia | Creates the Referral Attribution record at sign-up from the captured portal and/or the self-reported "how did you hear" answer, degrading to unknown on either gap |
| FEAT-33.SPEC-005 | Referral Data Access Restriction | Logic/Rule | Nadia, Dana | Enforces that no persona browses individual referral records inside the product; attribution is exposed only in aggregate to the growth metric |

Frontmatter counts match this table: spec_count: 5, screen_count: 1, automation_count: 2, logic_rule_count: 2, integration_count: 0, notification_count: 0.

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Discreet product mark | FEAT-33.SPEC-001 | Placement and rendering rule for the mark on every client-facing page and email | Phase 2 (Explicit) |
| Start from a portal | FEAT-33.SPEC-002, FEAT-33.SPEC-003, FEAT-33.SPEC-004 | Link capture on click, a landing page for viewers who decline to sign up, and attribution recorded once sign-up completes | Phase 2 (Explicit), elaborated in Phase 4 (Trigger-Response) |
| Tell us how you heard | FEAT-33.SPEC-004 | The self-reported answer is recorded into the Referral Attribution entity; the question itself is presented inside FEAT-20's onboarding screen (cross-feature -- see Cross-Feature Touchpoints) | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-33.SPEC-005 | Referral Data Access Restriction | Phase 5 (Rule-Constraint Discovery) | The Access field states plainly that "no persona browses referral data inside the product" and that Nadia is never told who signed up from her portal; this authorization/visibility constraint governs both SPEC-004's output and every role's access, so it needed its own rule spec rather than a buried note |
| FEAT-33.SPEC-002 | Referral Link Capture | Phase 4 (Trigger-Response Analysis) | The feature description states the referring portal is recorded but never states the mechanism; following a link and later completing sign-up are separate events, so a capture-and-carry automation is required between them |

## Entity-Lifecycle Coverage Matrix

**Entity: Referral Attribution**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-33.SPEC-004 | Created once, at sign-up, from the captured referring portal (SPEC-002) and/or the self-reported source answer (asked in FEAT-20); either half may be "unknown" | Matches the dependency map's lifecycle line: "Created by FEAT-33 at sign-up (answer captured in FEAT-20)" |
| Read (single) | N/A | No screen in this feature (or elsewhere) exposes an individual Referral Attribution record | Explicit product decision, not a gap -- see FEAT-33.SPEC-005 and the Access field: "No persona browses referral data inside the product" |
| Read (list) | N/A | No browsable list of referral records exists; the entity is read only in aggregate to compute the Growth Through Referral metric in success-metrics.md | Explicit product decision -- see FEAT-33.SPEC-005; aggregation is a success-metrics reporting concern, not a feature screen |
| Update | N/A | The dependency map states the entity is "Never updated" | Explicit non-goal, sourced from feature-dependency-map.md's Referral Attribution lifecycle line |
| Delete/Archive | N/A within this feature | Hard-deleted by FEAT-24 as part of full account deletion, per the dependency map's lifecycle line ("Deleted by FEAT-24 with the account"); no restore path and no independent retention window apply -- the record's lifetime is bound entirely to the owning Freelancer Account's lifetime | Cross-feature ownership, not a gap -- see Cross-Feature Touchpoints |
| State Transition | N/A | The entity carries no status field and passes through no states after creation (product-features.md, Data Notes: recorded_at only, never edited) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Freelancer Account | FEAT-33.SPEC-002 | Resolves which freelancer's portal a followed link belongs to, to capture the referring-portal reference |
| Freelancer Account | FEAT-33.SPEC-004 | Attaches the new Referral Attribution record to the Freelancer Account being created at sign-up |
| Branding Profile | FEAT-33.SPEC-001 | Confirms the mark's placement never overrides the freelancer's logo or brand colour (XBR-31) |
| Notification | FEAT-33.SPEC-001 | Confirms the mark's footer placement inside the emails FEAT-14 already sends, rather than this feature composing its own message |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Visitor follows the "Made with Clientroom" mark | Capture the referring freelancer's portal identifier for the session and fire the referral_mark_clicked signal | Standalone Automation | FEAT-33.SPEC-002 |
| New freelancer completes sign-up (FEAT-20) | Create the Referral Attribution record from the captured referring portal and the self-reported source answer, firing signup_attributed_to_portal and/or signup_source_answered | Standalone Automation | FEAT-33.SPEC-004 |
| Attribution cannot be recorded (e.g., the captured portal reference has expired or is missing) | Sign-up continues normally; the source is recorded as unknown rather than blocking account creation | Inline in triggering automation | FEAT-33.SPEC-004 |
| Anyone attempts to browse or open an individual referral record inside the product | The capability is not shown; only the aggregate growth figure exists, in success-metrics.md's reporting | Standalone Logic/Rule | FEAT-33.SPEC-005 |
| Freelancer account is deleted | Referral Attribution record is deleted with the account (no independent retention) | Cross-feature | FEAT-24 responsibility |

## Shared Context

**Shared Entities:**
- Referral Attribution -- created by SPEC-004 from data captured by SPEC-002 and the onboarding answer owned by FEAT-20; its visibility is restricted by SPEC-005. Fields: referring_portal (reference to a Freelancer Account, or unknown), self_reported_source (free-text answer, or unknown), recorded_at (required, immutable).
- Freelancer Account -- read (not owned) by SPEC-002 to resolve the referring portal and by SPEC-004 to attach the new attribution record to the freelancer being created; owned end-to-end by FEAT-20 (create) and FEAT-21 (update).

**Shared UI Patterns:**
- The mark itself (SPEC-001) is a single small, consistent footer element reused verbatim across every FEAT-05 portal page and every FEAT-14 email -- Spec Writers for both surfaces should treat its rendering rule as authored once here and referenced, not restated per surface.
- The Referral Landing Page (SPEC-003) and the sign-up entry point it links to (FEAT-20) share the same "return to portal in one step" affordance for a visitor who arrived via the mark but has a portal to return to.

**Shared Validation:**
- SPEC-005 defines the access restriction that SPEC-004's output must respect; no other spec in this feature re-derives who may view attribution data.

## Internal Dependency Map

```
SPEC-001 (Referral Mark Display) -> [visitor clicks the mark on a portal page or email] -> SPEC-002 (Referral Link Capture)
SPEC-002 (Referral Link Capture) -> [visitor chooses to sign up] -> FEAT-20 (Onboarding / First-Run Setup) sign-up flow
SPEC-002 (Referral Link Capture) -> [visitor declines to sign up] -> SPEC-003 (Referral Landing Page)
SPEC-003 (Referral Landing Page) -> [client contact returns to the portal] -> FEAT-05 (Client Portal Access)
SPEC-004 (Referral Attribution Recording) -> [reads the portal captured by] -> SPEC-002 (Referral Link Capture)
SPEC-004 (Referral Attribution Recording) -> [reads the self-reported answer from] -> FEAT-20 (Onboarding / First-Run Setup)
SPEC-004 (Referral Attribution Recording) -> [output visibility governed by] -> SPEC-005 (Referral Data Access Restriction)
```

**Default Entry:** SPEC-001 (Referral Mark Display) -- this feature has no in-product screen a user navigates to directly; it is entered only by a visitor following the mark that SPEC-001 places on FEAT-05 portal pages and FEAT-14 emails.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-33.SPEC-001 | Outbound | FEAT-05 (Client Portal Access [Magic-Link Login]) | The mark renders at the foot of every client-facing portal page | Portal page renders |
| FEAT-33.SPEC-001 | Outbound | FEAT-14 (Notifications [Email]) | The mark renders at the foot of every client-facing email | Email is composed for sending |
| FEAT-33.SPEC-001 | Inbound | FEAT-19 (Freelancer Branding) | The mark's rendering rule never overrides the freelancer's logo or brand colour (XBR-31) | Every client-facing render |
| FEAT-33.SPEC-002 | Outbound | FEAT-20 (Onboarding / First-Run Setup) | The captured referring portal is carried into the sign-up and first-run flow | Visitor who followed the mark chooses to sign up |
| FEAT-33.SPEC-003 | Outbound | FEAT-05 (Client Portal Access [Magic-Link Login]) | One-step return to the portal from the public landing page | Client contact taps "back to portal" |
| FEAT-33.SPEC-004 | Inbound | FEAT-20 (Onboarding / First-Run Setup) | The optional "How did you hear about us?" answer, asked during onboarding, is passed to attribution recording | New freelancer answers or skips the question |
| FEAT-33 (Referral Attribution entity) | Outbound | FEAT-24 (Data Export & Account Deletion) | The Referral Attribution record is deleted when the owning Freelancer Account is deleted | Freelancer requests account deletion |

## Non-Functional Notes

**Data volumes / growth:** One Referral Attribution record is created per new freelancer sign-up at most (never per portal view or per mark click) -- volume tracks the sign-up rate, not portal traffic; no distinct volume expectation beyond the freelancer base itself is set in assumptions-constraints.md.

**Responsiveness:** The mark and the Referral Landing Page are static, lightweight surfaces reached from a client-facing page a visitor is often viewing on a phone; they fall under ASMP-21's general expectation that client-facing pages become interactive within roughly 2 seconds on a typical mobile connection (assumptions-constraints.md, Non-Functional Expectations).

**Data sensitivity / privacy:** Low personal data -- a link between two Freelancer Accounts and a free-text answer, never client or project data (feature-dependency-map.md, Referral Attribution Data Sensitivity line); used only in aggregate, and the referring freelancer is never told who signed up (product-features.md, Access).

**Compliance flags:** GDPR-class handling applies because the record references personal Freelancer Account data (ASMP-24); it carries no independent retention window and is deleted entirely when the referencing account is deleted (FEAT-24), so no separate purge policy is needed for this feature.

## Non-Goals

- **An in-product referral analytics view for the referring freelancer** -- Excluded per product-features.md, Access: "Nadia is never told who signed up from her portal, and attribution is used only in aggregate to measure the growth loop." SPEC-005 enforces this as a rule rather than as an absent screen.
- **Hiding the mark on any current plan** -- Excluded per product-features.md, Validation & Limits: "The mark is shown on every plan in MVP and cannot be hidden." Whether paid plans may hide it later is an open pricing question the brief defers (BRIEF.md, Open Questions -- pricing), not a decision this feature can make.
- **Any exposure of client, project, or freelancer portal content through the mark or landing page** -- Excluded per scope-boundaries.md (SC-03): public or anonymous portal access is out of scope, and the mark leads only to a public product page, never to portal content.
- **A localized or multi-language public landing page** -- Excluded per scope-boundaries.md (SC-20): the product is English-only at launch, so SPEC-003's landing page carries no localization behavior.
- **A native-app deep link for the referral mark** -- Adjacency exclusion per scope-boundaries.md (SC-06): the product ships no native apps, so the mark and landing page are web-only, reached through the mobile browser like every other client-facing surface.



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



# Automation Spec: Referral Link Capture

## Overview

**Name:** Referral Link Capture
**ID:** FEAT-33.SPEC-002
**Type:** Automation
**Purpose:** Captures the referring freelancer's portal identifier the moment a visitor follows the "Made with Clientroom" mark, and carries that reference forward toward sign-up.
**Parent Feature:** FEAT-33 -- Portal Referral Attribution

## Scope and Non-Goals

**In Scope:**
- Capturing which Freelancer Account's portal page or email the mark was followed from
- Holding that reference for the length of the visit, scoped to the visitor's own session
- Routing the visitor to the Referral Landing Page (FEAT-33.SPEC-003) immediately after the click
- Carrying the captured reference forward into sign-up (FEAT-20.SPEC-001) if the visitor proceeds there, so FEAT-20.SPEC-004 can hand it off for recording
- Emitting `referral_mark_clicked` for every follow of the mark

**Non-Goals:**
- Creating the Referral Attribution record -- excluded per the dependency map's lifecycle line ("Created by FEAT-33 at sign-up"); this automation only captures and carries the reference, FEAT-33.SPEC-004 persists it once sign-up completes
- Rendering the mark itself -- owned by FEAT-33.SPEC-001; this automation begins only once the mark has already been followed
- Displaying the landing page's content -- owned by FEAT-33.SPEC-003; this automation only routes the visitor there
- Retaining the captured reference beyond the visit if the visitor never signs up -- excluded per the Non-Functional Notes' data-volume line ("volume tracks the sign-up rate, not portal traffic"); a click that never converts leaves no persisted record

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Visitor follows the mark on a client-facing portal page | FEAT-33.SPEC-001 (Referral Mark Display), rendered on FEAT-05 portal pages | Fires on every follow of the mark, regardless of the visitor's own sign-in state | The owning Freelancer Account reference for the portal page the mark rendered on |
| Visitor follows the mark in a client-facing email | FEAT-33.SPEC-001 (Referral Mark Display), rendered in FEAT-14 emails | Fires on every follow of the mark from an email | The owning Freelancer Account reference the email was composed for (FEAT-14.SPEC-001) |

## Processing Logic

1. Receive the click on the mark, along with the Freelancer Account reference of the surface it appeared on (the portal's owning account, or the email's addressed-for account).
2. Capture that reference as the "referring portal" for this visit, held in a session-scoped context tied to the visitor's browser session -- not yet a persisted record.
3. Route the visitor to the Referral Landing Page (FEAT-33.SPEC-003), passing the captured reference along in that same session context.
4. Emit `referral_mark_clicked`, noting which surface type it was followed from (portal page or email).
5. If the visitor later reaches sign-up (FEAT-20.SPEC-001), either directly from the landing page's "Sign up" action or by navigating there afterward within the same session, carry the same captured reference forward so it is available to FEAT-20.SPEC-004's hand-off.
6. If the captured reference's session context has expired before sign-up is reached (the visitor's session lifetime, per Business Rules, has elapsed), proceed to sign-up with no referring-portal reference, rather than attempting to recover or re-derive one.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|-----------------|--------------------|
| Capture succeeds | The mark is followed and the owning Freelancer Account reference is captured | A session-scoped reference is held for this visit; no persisted record yet | None -- the visitor is simply taken to the Referral Landing Page as expected | FEAT-33.SPEC-003 (Referral Landing Page), FEAT-20.SPEC-001 (Sign-Up & Account Creation) |
| Capture succeeds and carries through to sign-up | The visitor proceeds to sign-up within the reference's session lifetime | The reference is passed into FEAT-20.SPEC-001 and onward to FEAT-20.SPEC-004 | None -- invisible to the visitor; the sign-up form behaves identically with or without a captured reference | FEAT-20.SPEC-001, FEAT-20.SPEC-004, FEAT-33.SPEC-004 |
| Capture fails or the owning Freelancer Account no longer resolves (e.g., deleted, FEAT-24) | The mark's underlying account reference cannot be resolved at click time | No reference is captured for this visit | None -- the visitor still reaches the Referral Landing Page normally; the eventual sign-up simply carries no referring-portal reference | FEAT-33.SPEC-003, FEAT-33.SPEC-004 (records "unknown") |
| Captured reference expires before sign-up | The session-scoped reference's lifetime (platform parameter: `referral-capture-session-window`) elapses before the visitor reaches sign-up | The reference is discarded | None -- sign-up proceeds normally with no referring-portal reference, identical to a visitor who never followed a mark | FEAT-20.SPEC-001, FEAT-33.SPEC-004 (records "unknown") |

## Data Model

**Reads:** Freelancer Account -- reads only the account reference of the surface the mark was rendered on (the portal's owner, or the email's addressee-freelancer), to resolve which portal referred the visit. No other Freelancer Account fields are read.
**Creates:** None -- this automation holds a session-scoped reference only; it creates no persisted record. The persisted Referral Attribution record is created later, by FEAT-33.SPEC-004.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Capture is best-effort and non-blocking: a failure to resolve the referring account never prevents the visitor from reaching the Referral Landing Page or, later, signing up.
- The captured reference is held only for the length of the visit, scoped to the visitor's own session, and expires after platform parameter: `referral-capture-session-window` with no further activity -- it is never persisted independently of a completed sign-up.
- Only one referring-portal reference is held per visitor session at a time; following a second mark during the same session replaces the first (last-click-wins), since a sign-up can be attributed to at most one referring portal (dependency map: Referral Attribution's `referring_portal` is a single optional reference).
- This automation never re-derives or infers a referring portal from any other signal (browsing history, IP address, or similar) -- the reference is captured only from an explicit mark follow, per XBR-32's scope.

## Edge Cases

- **The mark's underlying Freelancer Account has since been deleted (FEAT-24) between the mark's earlier render and the visitor's click** -- Capture fails to resolve; the visitor still reaches the Referral Landing Page, and any later sign-up records the referring portal as unknown.
- **The visitor follows the mark, closes the browser, and returns later in a new, unrelated session** -- The earlier session-scoped reference is gone; the new session carries no referring-portal reference, and a subsequent sign-up in that new session records it as unknown.
- **The visitor follows a mark on Portal A, then later in the same session follows a mark on Portal B before signing up** -- The reference is overwritten: Portal B is what is captured and eventually attributed, per the last-click-wins rule.
- **The visitor follows the mark from an email that was addressed to a client contact but opened by someone else who forwarded it** -- Capture still resolves to the email's addressed-for Freelancer Account, since the reference is tied to the surface's own composition context, not to who personally clicks it.
- **Concurrent trigger firing (two different visitors follow marks on different portals at the same time)** -- Each fires its own independent capture, scoped to its own visitor session; neither affects the other.
- **Trigger fires while a previous run is in flight (the same visitor double-clicks the mark rapidly)** -- The second click's capture simply re-captures the same reference (or, if it was a different mark, overwrites per last-click-wins); there is no queued or conflicting state, since each capture is a single, immediate session write.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-33.SPEC-001 (Referral Mark Display) | Triggered by (inbound) | Following the rendered mark, on either a portal page or an email, fires this capture |
| FEAT-33.SPEC-003 (Referral Landing Page) | Affects (outbound) | Every successful click routes the visitor here immediately after capture |
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Affects (outbound) | The captured reference, when present, is carried forward into sign-up entry |
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | References (outbound) | Supplies the referring-portal reference this hand-off later passes to FEAT-33.SPEC-004 |
| FEAT-33.SPEC-004 (Referral Attribution Recording) | Affects (outbound) | The reference this automation captures (or its absence) is what FEAT-33.SPEC-004 eventually records, via FEAT-20.SPEC-004's hand-off |

## Analytics and Success Signals

- **referral_mark_clicked** (source surface: portal page / email) -- supports success-metrics.md: "Growth Through Referral"
- **referral_capture_expired** (had reached the landing page: yes/no) -- N/A -- no Stage 2 metric measures capture expiry directly; retained so a silently dropped capture is observable rather than invisible

## Acceptance Criteria

**FEAT-33.SPEC-002-AC-01:** Given Owen follows the "Made with Clientroom" mark on Nadia's portal home page, when this automation fires, then it captures Nadia's Freelancer Account as the referring portal for Owen's visit and routes him to the Referral Landing Page (FEAT-33.SPEC-003).

**FEAT-33.SPEC-002-AC-02:** Given a marketing lead follows the mark inside a client-facing email addressed by Nadia's account, when this automation fires, then it captures Nadia's Freelancer Account as the referring portal, identical to a portal-page follow.

**FEAT-33.SPEC-002-AC-03:** Given a visitor's referring-portal reference was captured, when the visitor proceeds from the Referral Landing Page to sign up (FEAT-20.SPEC-001), then the captured reference is carried forward unchanged into the sign-up entry.

**FEAT-33.SPEC-002-AC-04:** Given the mark's underlying Freelancer Account was deleted before the visitor's click resolves, when this automation attempts capture, then it fails silently, the visitor still reaches the Referral Landing Page, and no reference is carried forward.

**FEAT-33.SPEC-002-AC-05:** Given a visitor's captured reference has exceeded platform parameter: `referral-capture-session-window` with no sign-up yet, when the visitor reaches sign-up, then no referring-portal reference is carried forward.

**FEAT-33.SPEC-002-AC-06:** Given a visitor follows a mark on Portal A and later, in the same session, follows a mark on Portal B, when the visitor eventually signs up, then Portal B is the referring portal carried forward, not Portal A.

**FEAT-33.SPEC-002-AC-07:** Given every follow of the mark, when this automation fires, then `referral_mark_clicked` is emitted noting whether the source was a portal page or an email.

**FEAT-33.SPEC-002-AC-08:** Given a visitor closes the browser after following the mark and returns later in an unrelated session, when the visitor signs up in that new session, then no referring-portal reference is present.

**FEAT-33.SPEC-002-AC-09:** Given two different visitors follow marks on two different portals at the same time, when both captures fire, then each is scoped independently to its own visitor session with no interference.

**FEAT-33.SPEC-002-AC-10:** Given a visitor double-clicks the same mark rapidly, when the second click's capture fires, then it re-captures the same reference with no queued or conflicting state.

**FEAT-33.SPEC-002-AC-11:** Given a capture expires before the visitor reaches sign-up and the visitor had already reached the landing page, when the expiry occurs, then `referral_capture_expired` is emitted noting the visitor had reached the landing page.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



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



# Automation Spec: Referral Attribution Recording

## Overview

**Name:** Referral Attribution Recording
**ID:** FEAT-33.SPEC-004
**Type:** Automation
**Purpose:** Creates the Referral Attribution record once, at sign-up, from the referring portal captured by FEAT-33.SPEC-002 and the self-reported "how did you hear" answer captured by FEAT-20, degrading either half to unknown when it is missing.
**Parent Feature:** FEAT-33 -- Portal Referral Attribution

## Scope and Non-Goals

**In Scope:**
- Creating exactly one Referral Attribution record per new Freelancer Account, at the moment FEAT-20's hand-off (FEAT-20.SPEC-004) delivers the captured values
- Resolving the referring-portal reference at the moment of creation, treating an unresolvable reference (e.g., the referring account was deleted in the meantime) as unknown
- Recording the self-reported source exactly as received (already normalized to "unknown" by FEAT-20.SPEC-004 when skipped or blank)
- Emitting the signals that feed success-metrics.md's "Growth Through Referral" metric
- Guaranteeing the record is never created twice for the same Freelancer Account

**Non-Goals:**
- Asking the "How did you hear" question -- owned by FEAT-20.SPEC-002; this automation only receives the already-captured answer via FEAT-20.SPEC-004's hand-off
- Capturing the referring-portal reference from the mark click -- owned by FEAT-33.SPEC-002; this automation only receives that value, already resolved at the moment of the click, via the same hand-off
- Exposing the created record to any screen or persona -- excluded per product-features.md, Access ("No persona browses referral data inside the product"); FEAT-33.SPEC-005 is the authority for that restriction, and this automation creates the record with no read path of its own
- Updating or correcting the record after creation -- excluded per the dependency map's lifecycle line ("Never updated"); a Referral Attribution record is immutable from the moment it is created

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| FEAT-20's referral attribution hand-off completes | FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | Fires exactly once per new Freelancer Account, immediately after Nadia answers or skips the "How did you hear" question during onboarding | The new Freelancer Account reference, the self-reported source value (text or "unknown"), and the referring-portal reference (present or absent, originally captured by FEAT-33.SPEC-002) |

## Processing Logic

1. Receive the new Freelancer Account reference, the self-reported source value, and the referring-portal reference from FEAT-20.SPEC-004's hand-off.
2. Check whether a Referral Attribution record already exists for this Freelancer Account; if one does, take no further action (idempotency guard -- see Business Rules).
3. If a referring-portal reference is present, verify it still resolves to an existing Freelancer Account. If it does not resolve (for example, the referring account was deleted between the mark click and this sign-up completing), treat the reference as absent.
4. Create a Referral Attribution record for the new Freelancer Account with: `referring_portal` set to the resolved reference, or "unknown" if absent or unresolvable; `self_reported_source` set to the received value (already "unknown" if the question was skipped, per FEAT-20.SPEC-004); `recorded_at` set to the current time.
5. Emit `signup_attributed_to_portal` if a referring portal was recorded (not "unknown"), and emit `signup_source_answered` if a self-reported answer was recorded (not "unknown"). Always emit `referral_attribution_recorded` with both outcomes noted, regardless of whether either half is known, so the growth metric's full denominator of sign-ups is observable.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|-----------------|--------------------|
| Recorded with both known | A referring-portal reference resolved and a self-reported answer was given | Referral Attribution record created with both fields populated | None -- invisible to Nadia; her onboarding already advanced in FEAT-20.SPEC-002 | FEAT-33.SPEC-005 (governs the new record's visibility) |
| Recorded with only self-reported known | No referring-portal reference resolved, but a self-reported answer was given | Record created with `referring_portal` set to "unknown" | None | FEAT-33.SPEC-005 |
| Recorded with only referring portal known | A referring-portal reference resolved, but the question was skipped (self-reported source is "unknown") | Record created with `self_reported_source` set to "unknown" | None | FEAT-33.SPEC-005 |
| Recorded fully unknown | Neither a referring-portal reference nor a self-reported answer is available | Record still created (`recorded_at` is always required) with both fields "unknown" | None | FEAT-33.SPEC-005 |
| Recording fails | This automation cannot complete (e.g., a transient failure creating the record) | No Referral Attribution record exists for this account | None -- the freelancer's sign-up and onboarding are already complete and fully unaffected; the growth metric simply undercounts this one sign-up | FEAT-20 (unaffected) |

## Data Model

**Reads:** Freelancer Account -- reads the new account's reference to attach the record to it, and reads the referring-portal reference (another Freelancer Account) only to confirm it still resolves to an existing account.
**Creates:** Referral Attribution -- `referring_portal` (reference to the referring Freelancer Account, or "unknown"), `self_reported_source` (free-text answer, or "unknown"), `recorded_at` (required).
**Updates:** None -- a Referral Attribution record is never updated after creation, per the dependency map's lifecycle line.
**Deletes:** None directly -- the record is later deleted only as part of FEAT-24's account-deletion cascade, outside this automation's scope.

## Business Rules

- This automation fires and creates a record at most once per Freelancer Account -- a second hand-off for the same account (should one somehow occur) is a no-op, since the idempotency check in Processing Logic finds an existing record and takes no further action.
- Both fields degrade independently to "unknown" -- the absence of one never affects the recording of the other (referring_portal and self_reported_source are recorded exactly as received, with no cross-field dependency).
- Recording is non-blocking, mirroring FEAT-20.SPEC-004's own rule: a failure here never reopens, delays, or re-surfaces anything to the freelancer's completed onboarding.
- XBR-32: attribution is used only in aggregate to measure the growth loop -- this automation creates the record but never exposes it to any screen; FEAT-33.SPEC-005 governs that restriction.
- The record's visibility, once created, is governed entirely by FEAT-33.SPEC-005 -- this automation does not re-derive or restate who may access it.

## Edge Cases

- **The referring Freelancer Account is deleted (FEAT-24) between the mark click and this sign-up completing** -- The reference fails to resolve at creation time and is recorded as "unknown," per Processing Logic step 3.
- **FEAT-20.SPEC-004's hand-off is somehow delivered twice for the same account (e.g., a retried delivery)** -- The idempotency check finds the existing record and takes no action; no second record is ever created.
- **The self-reported source contains only whitespace** -- Not possible at this automation's boundary: FEAT-20.SPEC-004 already normalizes a whitespace-only answer to "unknown" before handing it off, so this automation only ever receives a defined value.
- **Concurrent trigger firing (two different new freelancers complete sign-up at effectively the same time)** -- Each fires its own independent recording for its own Freelancer Account; there is no shared state between them, and no race exists since each account can have only its own record.
- **Trigger fires while a previous run is in flight for the same account** -- Cannot occur: FEAT-20.SPEC-004 fires this hand-off at most once per account (its own once-only rule), so no second run for the same account can start while the first is in flight.
- **The Referral Attribution record creation itself fails after Freelancer Account creation already succeeded** -- The Freelancer Account remains fully created and usable; only the attribution record is missing, silently undercounting that sign-up in the growth metric, consistent with the non-blocking business rule.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | Triggered by (inbound) | Delivers the Freelancer Account reference, self-reported source, and referring-portal reference |
| FEAT-33.SPEC-002 (Referral Link Capture) | References (inbound) | Originates the referring-portal reference this automation resolves and records |
| FEAT-20 (Onboarding / First-Run Setup) | References (inbound) | Originates the self-reported "how did you hear" answer, asked in FEAT-20.SPEC-002 |
| FEAT-33.SPEC-005 (Referral Data Access Restriction) | Affects (outbound) | Governs who may access the record this automation creates -- no one, by product decision |
| FEAT-24 (Data Export & Account Deletion) | References (outbound) | The record created here is deleted with the owning Freelancer Account, with no independent retention window |

## Analytics and Success Signals

- **signup_attributed_to_portal** (referring portal known: yes) -- supports success-metrics.md: "Growth Through Referral"
- **signup_source_answered** (self-reported source known: yes) -- supports success-metrics.md: "Growth Through Referral"
- **referral_attribution_recorded** (referring portal known: yes/no; self-reported source known: yes/no) -- supports success-metrics.md: "Growth Through Referral" (this event always fires, giving the metric its full sign-up denominator alongside the two conditional numerator events above)

## Acceptance Criteria

**FEAT-33.SPEC-004-AC-01:** Given a new freelancer's account was referred by a resolvable portal and she also answered the "How did you hear" question, when this automation fires, then the Referral Attribution record is created with both `referring_portal` and `self_reported_source` populated, and both `signup_attributed_to_portal` and `signup_source_answered` are emitted.

**FEAT-33.SPEC-004-AC-02:** Given a new freelancer arrived with no referring-portal reference but answered the question, when this automation fires, then the record is created with `referring_portal` set to "unknown" and `self_reported_source` populated, and only `signup_source_answered` fires among the two conditional events.

**FEAT-33.SPEC-004-AC-03:** Given a new freelancer arrived via a resolvable referring portal but skipped the question, when this automation fires, then the record is created with `self_reported_source` set to "unknown" and `referring_portal` populated, and only `signup_attributed_to_portal` fires among the two conditional events.

**FEAT-33.SPEC-004-AC-04:** Given a new freelancer arrived with neither a referring portal nor an answer, when this automation fires, then the record is still created with both fields set to "unknown," and `referral_attribution_recorded` fires noting both as unknown, with neither conditional event firing.

**FEAT-33.SPEC-004-AC-05:** Given a referring-portal reference points to a Freelancer Account that was deleted before this sign-up completed, when this automation resolves the reference, then it is recorded as "unknown" rather than causing an error.

**FEAT-33.SPEC-004-AC-06:** Given a Referral Attribution record already exists for a Freelancer Account, when FEAT-20.SPEC-004's hand-off is somehow delivered again for that same account, then this automation takes no action and no second record is created.

**FEAT-33.SPEC-004-AC-07:** Given this automation's recording fails after the Freelancer Account was already created, when the failure occurs, then the Freelancer Account remains fully usable and no error is surfaced to the new freelancer.

**FEAT-33.SPEC-004-AC-08:** Given this automation successfully creates a Referral Attribution record, when the creation completes, then `referral_attribution_recorded` is emitted noting whether each of the two fields was known or unknown.

**FEAT-33.SPEC-004-AC-09:** Given two different new freelancers complete sign-up at effectively the same time, when this automation fires for each, then each creates its own independent record with no shared state or race between them.

**FEAT-33.SPEC-004-AC-10:** Given a Referral Attribution record has been created for a Freelancer Account, when any process attempts to update it later, then no update path exists -- the record remains exactly as recorded at creation.

**FEAT-33.SPEC-004-AC-11:** Given a Freelancer Account whose Referral Attribution record was created, when that account is later deleted (FEAT-24), then the Referral Attribution record is deleted with it, with no independent retention.

**FEAT-33.SPEC-004-AC-12:** Given this automation creates a record with an unknown referring portal, when FEAT-33.SPEC-005 is later consulted for that record's visibility, then no persona -- including Nadia and Dana -- has any path to view the individual record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Referral Data Access Restriction

## Overview

**Name:** Referral Data Access Restriction
**ID:** FEAT-33.SPEC-005
**Type:** Logic/Rule
**Purpose:** Enforces that no persona browses an individual Referral Attribution record inside the product -- attribution exists only as an aggregate figure for the Growth Through Referral metric, and the referring freelancer is never told who signed up from her portal.
**Parent Feature:** FEAT-33 -- Portal Referral Attribution
**Governed Entity:** Referral Attribution

## Scope and Non-Goals

**In Scope:**
- Authorization rules for every action on the Referral Attribution record, for every role in the Access Matrix
- The explicit statement that the record is never queried for display by any screen, including Dana's otherwise broad, read-only support access (FEAT-31)
- The boundary between this restriction and the aggregate Growth Through Referral figure, which is a success-metrics.md reporting concern outside any in-product screen
- Confirming field-level coverage of the governed entity, even though no field carries its own validation rule (validation is owned by the creating automation, FEAT-33.SPEC-004)

**Non-Goals:**
- Validating or deriving the Referral Attribution record's field values -- owned by FEAT-33.SPEC-004 (Referral Attribution Recording), which creates the sole record and sets every field at that moment; this spec governs only who may subsequently access it, never how it is populated
- Whether the record appears inside a full account data export (FEAT-24) -- FEAT-24 owns the scope and content of what a freelancer's own data export contains; this spec restricts only in-product browsing of an individual record, per product-features.md's exact language, and does not extend to an out-of-product export file
- Defining the aggregate Growth Through Referral figure's calculation -- owned by success-metrics.md as a Stage 2 reporting concern; this spec only draws the boundary that no in-product screen surfaces that aggregate to any persona
- Deleting the record -- owned entirely by FEAT-24's account-deletion cascade (dependency map: "Deleted by FEAT-24 with the account"); no persona-initiated deletion path exists for this spec to restrict

## Governed Entity

**Entity:** Referral Attribution
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| referring_portal | derived (reference, or "unknown") | Reference to the referring freelancer's Freelancer Account, or "unknown" when no portal referred the sign-up or the reference could not be resolved |
| self_reported_source | text (or "unknown") | The new freelancer's optional free-text answer to "How did you hear about us?", or "unknown" when skipped |
| recorded_at | date (required) | Timestamp of the record's one-time creation |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-33.SPEC-004 (Referral Attribution Recording) | Portal Referral Attribution | At creation -- the record is written once and exposed to no subsequent read path from any screen or automation this spec governs |
| FEAT-31 (Operator Support Access, referenced) | Operator Support Access | Structural exclusion -- Dana's otherwise broad, read-only support session (View across most entities per the Access Matrix) explicitly excludes Referral Attribution; no support-session screen queries this entity |
| success-metrics.md (Growth Through Referral, referenced) | -- (Stage 2 reporting artifact, not an in-product spec) | The only "read" of this entity anywhere -- an aggregate figure computed and reported outside the product's own screens, never surfaced to any persona through a screen |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| referring_portal | No validation beyond data type -- set once at creation by FEAT-33.SPEC-004; this spec governs access to the field, not its population | Always | -- | -- | -- |
| self_reported_source | No validation beyond data type -- set once at creation by FEAT-33.SPEC-004; this spec governs access to the field, not its population | Always | -- | -- | -- |
| recorded_at | No validation beyond data type -- set automatically at creation and never re-checked, since the record is never edited | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Independent degradation | referring_portal, self_reported_source | Neither field's value (known or "unknown") depends on or constrains the other -- a record may hold any of the four combinations of known/unknown across the two fields | N/A -- this is a creation-time rule owned by FEAT-33.SPEC-004; this spec only confirms no access rule in this spec depends on either field's value |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a Referral Attribution record | -- (no persona) | Never via any persona-initiated action -- creation happens only through FEAT-33.SPEC-004's own automated processing at sign-up | N/A -- no screen or control anywhere in the product offers any persona a way to create this record directly |
| View an individual Referral Attribution record | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Never, for all four roles, always | No screen, list, or detail view anywhere in the product exposes an individual record. Nadia specifically is never told who signed up from her portal -- there is no "who did I refer" screen in Settings, the Dashboard, or anywhere else. Dana's otherwise broad support-session read access (FEAT-31) does not extend to this entity; no support screen shows it |
| View the aggregate Growth Through Referral figure | -- (no persona) | Never, via any in-product screen -- the aggregate is a success-metrics.md reporting artifact consumed outside the product's own screens, not a capability exposed to any persona | No screen in the product -- including any Nadia sees on her Dashboard (FEAT-12) or Settings (FEAT-21) -- surfaces this aggregate to her or to any other role |
| Update a Referral Attribution record | -- (no persona) | Never -- the record is immutable from the moment of creation (dependency map: "Never updated") | N/A -- no edit control exists for any persona, since the record has no fields exposed to any screen in the first place |
| Delete a Referral Attribution record | -- (no persona) | Never, directly -- deletion happens only as part of FEAT-24's account-deletion cascade, not as a persona-initiated action within this feature | N/A -- no delete control exists for any persona; the only removal path is the whole-account cascade FEAT-24 owns |

## Defaults and Derivations

No defaults or derivations are governed by this spec. Field defaults and derivations belong entirely to FEAT-33.SPEC-004, which creates the sole record for each Freelancer Account and sets every field's value exactly once, at that moment. This spec governs only who may subsequently read or act on the entity, never how its values are populated.

## Business Rules

- XBR-32: attribution is used only in aggregate to measure the growth loop -- this spec is XBR-32's authority for the access-restriction half of that rule; FEAT-33.SPEC-004 is its authority for how the record is created.
- The dependency map's Referral Attribution lifecycle line states it is "Read by FEAT-33 (aggregate growth measurement only)" -- no other feature, screen, or automation in the product reads this entity for any other purpose, including features that otherwise have broad read access to a freelancer's data (FEAT-13's Activity & Audit Trail, FEAT-31's Operator Support Access).
- Dana's support session (FEAT-31) is otherwise read-only across most of a freelancer's data, per the Access Matrix's "View" entries for her row -- Referral Attribution is a deliberate, explicit exception to that breadth, not an oversight.
- This restriction applies from the moment of creation onward -- there is no transitional period, retroactive exposure, or admin override anywhere in the product definition that surfaces an individual record to any persona.

## Edge Cases

- **Dana opens a support session (FEAT-31) for a freelancer whose portal referred a new sign-up** -- The Referral Attribution record created by that sign-up is not shown anywhere in the support session, even though most of the freelancer's other data is visible to Dana in that same session.
- **Nadia looks for a "who referred this signup" or "my referrals" screen anywhere in Settings, the Dashboard, or Client & Project Management** -- No such screen exists anywhere in the product; there is no control to look for.
- **Someone attempts to reach a direct record identifier or URL for a specific Referral Attribution record** -- Not applicable: no such route exists in the product, since no screen ever renders, links to, or exposes an individual record's identifier.
- **The Growth Through Referral aggregate would, in principle, be derivable by cross-referencing Freelancer Account creation dates with any exposed referral data** -- Not a concern in practice, since no referral data of any kind (individual or partial) is exposed through any screen for any persona to cross-reference in the first place.
- **A future feature request asks for an in-product "who signed up from my portal" view** -- Out of scope for this spec and for the current product definition; product-features.md's Access field and this spec's Authorization Rules table would both need to change through a Brief update, which is outside any spec's authority to make unilaterally.

## Acceptance Criteria

**FEAT-33.SPEC-005-AC-01:** Given Nadia is signed in and looks anywhere in Settings, her Dashboard, or Client & Project Management for a way to see who signed up from her portal, when she searches those areas, then no such screen or control exists anywhere.

**FEAT-33.SPEC-005-AC-02:** Given Owen is signed in to his client portal, when he looks for any referral-related data about other freelancers or accounts, then no such capability is shown -- his access to Portal Referral relates only to viewing the mark itself (FEAT-33.SPEC-001), never to attribution data.

**FEAT-33.SPEC-005-AC-03:** Given Priya is signed in to her client portal, when she looks for referral attribution data, then the same result as Owen's applies -- no such capability exists for her role either.

**FEAT-33.SPEC-005-AC-04:** Given Dana opens a read-only support session (FEAT-31) for a freelancer's account that has an associated Referral Attribution record, when she reviews the account's data during that session, then the Referral Attribution record is not shown, even though her session otherwise reads most of that account's data.

**FEAT-33.SPEC-005-AC-05:** Given a Referral Attribution record exists, when any persona attempts to reach a direct view of it (by any means the product exposes), then no route or screen exists to do so.

**FEAT-33.SPEC-005-AC-06:** Given the Growth Through Referral metric is computed in aggregate, when any persona views any in-product screen (Dashboard, Settings, or otherwise), then that aggregate figure is not surfaced there -- it exists only as a success-metrics.md reporting artifact outside the product's own screens.

**FEAT-33.SPEC-005-AC-07:** Given a Referral Attribution record has been created, when any process attempts to create a second one for the same Freelancer Account through any persona-facing control, then no such control exists -- creation is exclusively FEAT-33.SPEC-004's own automated action.

**FEAT-33.SPEC-005-AC-08:** Given a Referral Attribution record exists, when any persona attempts to edit or delete it directly (outside FEAT-24's account-deletion cascade), then no edit or delete control exists anywhere in the product for any role.

**FEAT-33.SPEC-005-AC-09:** Given every field in the Referral Attribution entity, when this spec's Field Validation Rules are reviewed, then each field is explicitly addressed as "no validation beyond data type," confirming none was accidentally skipped.

**FEAT-33.SPEC-005-AC-10:** Given a Freelancer Account with an associated Referral Attribution record is deleted through FEAT-24, when the deletion completes, then the Referral Attribution record is removed with it -- the only removal path this spec recognizes for the entity.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 0 (explicitly N/A) | 0 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
