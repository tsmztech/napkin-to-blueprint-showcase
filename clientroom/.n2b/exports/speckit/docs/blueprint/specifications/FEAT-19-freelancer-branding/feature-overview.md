---
document_type: feature-overview
feature_number: FEAT-19
feature_name: Freelancer Branding
feature_slug: freelancer-branding
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 3
screen_count: 1
automation_count: 0
logic_rule_count: 2
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Freelancer Branding

## Summary

**Feature:** Freelancer Branding
**ID:** FEAT-19
**Description:** The freelancer sets a logo and brand colour so every client-facing screen looks like her own practice rather than a generic shared tool.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Ecosystem & Integrations: "each freelancer's own logo and colours on their portal." Ranked Important rather than Core because the product functions correctly, if generically, without it; phased MVP because BRIEF.md's growth loop depends on clients seeing a branded, professional portal from the very first project ("new freelancers arrive after a client or peer sees someone else's portal," BRIEF.md, Business Context). [RESEARCH-INFORMED: white-label branding is SuiteDash's most praised capability (review roundups citing G2, Capterra, AppSumo, HIGH), while HoneyBook users want more freedom to match their brand (G2, MEDIUM) — logo and colour on every client screen and email is the MVP answer, with a custom domain (FEAT-27) later] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Set a logo — upload the freelancer's own logo
- Set a brand colour — choose a primary colour applied across client-facing screens

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-19.SPEC-001 | Branding Settings | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia uploads a logo, picks a brand colour, and can reset to the neutral default; Dana views the same settings read-only inside a logged support session |
| FEAT-19.SPEC-002 | Branding Upload & Legibility Validation Rules | Logic/Rule | Nadia (Freelancer) | Governs logo file size/format limits, the one-logo/one-colour-per-account limit, and the legibility adjustment applied to a colour that would make text hard to read |
| FEAT-19.SPEC-003 | Branding Application & Fallback Rule | Logic/Rule | All | Governs how the freelancer's logo and colour (or the clean neutral default when unset) are applied to every client-facing screen and email, and how the referral mark coexists with them without overriding them |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Set a logo — upload the freelancer's own logo | FEAT-19.SPEC-001, FEAT-19.SPEC-002, FEAT-19.SPEC-003 | The settings screen captures the upload; the validation rule enforces size/format limits and the one-logo-per-account limit; the application rule puts the logo on every client-facing screen and email | Phase 2 (Explicit) |
| Set a brand colour — choose a primary colour applied across client-facing screens | FEAT-19.SPEC-001, FEAT-19.SPEC-002, FEAT-19.SPEC-003 | The settings screen captures the colour choice; the validation rule adjusts a colour that would harm legibility and tells Nadia; the application rule puts the colour on every client-facing screen and email | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-19.SPEC-002 | Branding Upload & Legibility Validation Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field names five distinct rules governing a single entity (logo size limit, logo format limit, one-logo-per-account, one-colour-per-account, colour legibility adjustment) — at or above the 5+ rule threshold for a standalone Logic/Rule spec, and the legibility adjustment is a non-trivial derivation (a contrast check with an automatic corrected value), not a simple field check |
| FEAT-19.SPEC-003 | Branding Application & Fallback Rule | Phase 4 (Trigger-Response Analysis / External Dependencies lens, cross-entity propagation) | The dependency map names this feature as the Authority for XBR-31 ("logo and colour apply to every client-facing screen and email... fall back to a clean neutral default... adjusted for legibility") — a cross-feature rule read by eight other features (FEAT-02, FEAT-05, FEAT-06, FEAT-09, FEAT-14, FEAT-27, FEAT-31, FEAT-33) that needed a standalone home rather than being left implicit on the settings screen |

## Entity-Lifecycle Coverage Matrix

**Entity: Branding Profile**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-19.SPEC-001 | The Branding Profile record is created the first time Nadia saves a logo or colour on the Branding Settings screen (whether reached directly or via FEAT-20's guided-setup step) | Before this first save, the profile is effectively absent and the feature behaves as its Empty state (neutral default) |
| Read (single) | FEAT-19.SPEC-001 | The settings screen loads the freelancer's current logo and colour (or the empty/default state) when opened by Nadia or Dana | Also read by FEAT-19.SPEC-003 whenever any client-facing screen or email needs to apply branding |
| Read (list) | N/A | There is exactly one Branding Profile per freelancer account (dependency map: "One per Freelancer Account"); no list view applies | — |
| Update | FEAT-19.SPEC-001 | Nadia changes her logo and/or colour, or resets to the neutral default, on the settings screen; a save from a second session simply replaces the first (dependency map, Contention: "last-write-wins") | Reset-to-default is an update that clears both fields back to unset, not a delete of the record |
| Delete/Archive | N/A | The dependency map states the Branding Profile is "Deleted by FEAT-24" (account deletion) — this feature never deletes the record itself, only sets or clears its two fields. This is an explicit non-goal: because the profile holds no sensitive data (Data Sensitivity: None) and is one-per-account, its deletion is tied to the account's own deletion lifecycle rather than given an independent retention/purge policy here | — |
| State Transition | N/A | The profile is a plain pair of optional values (logo, brand_colour) with no workflow states of its own; "unset" versus "set" is a rendering condition handled as the settings screen's Empty state (Phase 6), not an entity lifecycle transition | — |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Freelancer Account | FEAT-19.SPEC-001 | The settings screen ties the single Branding Profile to the signed-in freelancer's own account |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia uploads a logo and/or picks a colour and saves | Validate the logo's file size and format, enforce the one-logo/one-colour-per-account limit, and check the colour for legibility | Standalone Logic/Rule | FEAT-19.SPEC-002 |
| The chosen colour would make text hard to read | Adjust the colour for legibility and tell Nadia inline, on the same screen | Standalone Logic/Rule, surfaced inline in the triggering screen (no separate delivery channel or audience — stays inline per Phase 4's disposition rule) | FEAT-19.SPEC-002 / FEAT-19.SPEC-001 |
| A logo upload fails (bad file, size/format rejection, network interruption) | Preserve the previous branding (previous logo and colour stay active and applied) and show an inline error; nothing changes until a new upload succeeds | Inline in triggering screen (Error state) | FEAT-19.SPEC-001 |
| Nadia saves a valid logo and/or colour | Apply the new logo/colour to every client-facing screen and email immediately | Standalone Logic/Rule | FEAT-19.SPEC-003 |
| Nadia resets branding to default, or no branding has ever been set | Fall back to the clean, neutral default across every client-facing screen and email | Standalone Logic/Rule | FEAT-19.SPEC-003 |
| Nadia saves valid branding, resets to default, or an upload fails legibility/limits validation | Emit the corresponding signal (`branding_logo_uploaded`, `branding_color_set`, `branding_reset_to_default`) | Inline in triggering spec | FEAT-19.SPEC-001 |
| Onboarding guided setup (FEAT-20) reaches its "Set branding" step | Present this feature's Branding Settings screen inline as a skippable step; skipping leaves the neutral default in place with no lost progress | Cross-feature | FEAT-20 responsibility, entry point is FEAT-19.SPEC-001 |
| Nadia returns to Settings (FEAT-21) to finish a skipped branding step | Navigate to the Branding Settings screen, pre-populated with whatever was set (or still empty) | Cross-feature | FEAT-21 responsibility, entry point is FEAT-19.SPEC-001 |
| Dana opens a read-only support session on the freelancer's account (FEAT-31) | Render the Branding Settings screen with no save control — view-only | Inline in triggering screen (Permission Denied-adjacent: read-only rather than hidden, per the Access Matrix's "View" row for Support Access) | FEAT-19.SPEC-001 |
| Owen or Priya attempts to reach the Branding Settings screen | Nothing is shown — the Access Matrix gives both roles "None" on Branding, Onboarding & Settings; client contacts only ever see the resulting branding applied elsewhere, never this screen | Inline in triggering screen (Permission Denied state) | FEAT-19.SPEC-001 |
| Any client-facing screen renders, or a client email is sent (FEAT-02, FEAT-05, FEAT-06, FEAT-09, FEAT-14) | Apply the freelancer's current logo/colour, or the neutral default, per the Branding Application & Fallback Rule | Standalone Logic/Rule, cross-feature | FEAT-19.SPEC-003 |
| The referral mark (FEAT-33, XBR-32) renders alongside branding on a client-facing page or email | Show the referral mark next to the applied branding without overriding it | Standalone Logic/Rule, cross-feature | FEAT-19.SPEC-003 |
| The freelancer account is deleted (FEAT-24) | The Branding Profile is deleted along with the account | Cross-feature | FEAT-24 responsibility |

## Shared Context

**Shared Entities:**
- Branding Profile — logo and brand_colour fields are created/read/updated by FEAT-19.SPEC-001, validated and legibility-adjusted by FEAT-19.SPEC-002, and read and applied (or defaulted) by FEAT-19.SPEC-003 wherever a client-facing screen or email needs branding. No other spec in this feature writes to it.

**Shared UI Patterns:**
- Clean neutral default — FEAT-19.SPEC-003 defines the single "no branding set" look (clean, professional, lots of white space, per BRIEF.md's Constraints) that every consuming feature's client-facing screens and emails fall back to. Spec Writers for FEAT-02, FEAT-05, FEAT-06, FEAT-09, and FEAT-14 should describe this identically rather than inventing a per-screen "unbranded" look.
- Preserve-on-failure — FEAT-19.SPEC-001's Error state (a failed upload keeps the previous branding active) is the one interaction pattern unique to this feature and should be described consistently wherever the upload control appears (settings, or embedded in FEAT-20's onboarding step).

**Shared Validation:**
- FEAT-19.SPEC-002 is the single source of truth for what counts as a valid logo and a legible colour. FEAT-19.SPEC-001 (screen-level save) applies it directly; FEAT-19.SPEC-003 only ever applies logo/colour values that have already passed FEAT-19.SPEC-002, never re-validating them.

## Internal Dependency Map

```
SPEC-001 (Branding Settings) -> [Nadia saves logo/colour] -> SPEC-002 (Branding Upload & Legibility Validation Rules)
SPEC-002 (Branding Upload & Legibility Validation Rules) -> [validation passes, colour adjusted if needed] -> SPEC-003 (Branding Application & Fallback Rule)
SPEC-001 (Branding Settings) -> [Nadia resets to default] -> SPEC-003 (Branding Application & Fallback Rule)
SPEC-003 (Branding Application & Fallback Rule) -> [reads current Branding Profile to render settings] -> SPEC-001 (Branding Settings)
```

**Default Entry:** SPEC-001 (Branding Settings) — the feature's only screen. It has no independent top-level navigation entry of its own; it is reached from FEAT-20's guided setup step or FEAT-21's Settings area (see Cross-Feature Touchpoints).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-19.SPEC-001 | Inbound | FEAT-20 (Onboarding / First-Run Setup) | Guided setup's "Set branding" step is a skippable entry point into this feature's settings screen | Nadia reaches Step 3 of Freelancer First-Time Setup |
| FEAT-19.SPEC-001 | Inbound | FEAT-21 (Settings & Account Management) | Nadia returns to complete a previously skipped branding step | Nadia navigates to Settings > Branding |
| FEAT-19.SPEC-001 | Outbound | FEAT-27 (Custom Domain per Freelancer, Later) | From branding settings, Nadia can go further and set up her own domain | Nadia opens custom domain setup (Later phase) |
| FEAT-19.SPEC-001 | Inbound | FEAT-31 (Operator Support Access) | Dana views the branding settings screen read-only inside a logged, time-limited support session | A support session is opened on the freelancer's account |
| FEAT-19.SPEC-001 | Inbound | FEAT-24 (Account Deletion) | The Branding Profile is deleted as part of account deletion | Freelancer account deletion completes |
| FEAT-19.SPEC-003 | Outbound | FEAT-02 (Proposal Creation & Sending) | Proposal screens render the freelancer's logo/colour or the neutral default | A proposal screen is viewed |
| FEAT-19.SPEC-003 | Outbound | FEAT-05 (Client Portal Access) | Every client portal screen renders the freelancer's logo/colour or the neutral default | The client portal is viewed |
| FEAT-19.SPEC-003 | Outbound | FEAT-06 (Deliverable Upload & Sharing) | Deliverable review screens render the freelancer's logo/colour or the neutral default | A deliverable screen is viewed |
| FEAT-19.SPEC-003 | Outbound | FEAT-09 (Invoice Generation & Sending) | Invoice screens render the freelancer's logo/colour or the neutral default | An invoice is viewed |
| FEAT-19.SPEC-003 | Outbound | FEAT-14 (Notifications — Email) | Every client email carries the freelancer's logo/colour or the neutral default (XBR-31) | A client email is sent |
| FEAT-19.SPEC-003 | Outbound | FEAT-33 (Portal Referral Attribution) | The referral mark sits alongside the applied branding without overriding it (XBR-32) | A client-facing page or email renders |

## Non-Functional Notes

**Data volumes / growth:** One Branding Profile per freelancer account, holding a single logo file and a single colour value; volume tracks the expected few thousand freelancers in year one (scope-boundaries.md, SC-21) and carries no growth concern of its own.

**Responsiveness:** Logo file size and format are limited specifically so client-facing pages stay fast on mobile (product-features.md, Validation & Limits), consistent with client-facing pages becoming interactive within roughly 2 seconds on a typical mobile connection (assumptions-constraints.md, ASMP-21); saving branding settings itself is a small configuration write with no perceptible wait (feature's own States field: "Loading: N/A — a small configuration form").

**Data sensitivity / privacy:** The logo and brand colour are public-facing brand assets with no personal data (feature-dependency-map.md, Entity: Branding Profile, Data Sensitivity: "None").

**Compliance flags:** A brand colour that would make text hard to read is automatically adjusted for legibility, and every client-facing screen must remain usable with a screen reader and keyboard and never rely on colour alone (assumptions-constraints.md, ASMP-27); no other compliance regime applies to this feature.

## Non-Goals

- **Multiple logos, multiple colours, or a full brand kit per account** — Limit stated directly in product-features.md's Validation & Limits: "one logo and one primary colour per freelancer account in v1." A single, simple brand identity is the deliberate MVP scope; anything richer is left for a later iteration.
- **Custom domain per freelancer** — Deferred per scope-boundaries.md's Deferral Notes: target phase Later, because BRIEF.md itself leaves the timing an open question and recommends prioritizing it "once branding (FEAT-19) adoption shows freelancers want to go further with their own domain." FEAT-27 owns this later capability; FEAT-19.SPEC-001 only navigates to it.
- **Client-side or team-side ability to change branding** — Per the Access Matrix (user-persona.md), Owen and Priya both carry "None" on Branding, Onboarding & Settings, and scope-boundaries.md SC-01 excludes any internal-staff or team-of-many account model that could otherwise justify a second branding editor. Branding remains Nadia's alone to set.
- **Automatic jurisdictional or brand-guideline compliance checking on the logo or colour** — Adjacency exclusion: the product definition's only stated check on the uploaded logo and colour is technical (file size/format) and accessibility-driven (legibility); the dependency map's Data Sensitivity for the Branding Profile is "None," and no assumptions-constraints.md entry asks this feature to validate trademark, brand-guideline, or content-appropriateness concerns.
