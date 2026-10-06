# FEAT-19 — Freelancer Branding

This chapter covers Freelancer Branding, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 3 specifications carrying 52 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-19.SPEC-001 | Branding Settings | screen | 24 |
| FEAT-19.SPEC-002 | Branding Upload & Legibility Validation Rules | logic-rule | 14 |
| FEAT-19.SPEC-003 | Branding Application & Fallback Rule | logic-rule | 14 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Branding Settings

## Overview

**Name:** Branding Settings
**ID:** FEAT-19.SPEC-001
**Type:** Screen
**Purpose:** Nadia uploads a logo, picks a brand colour, and can reset both to the neutral default; Dana views the same screen read-only inside a logged support session.
**Parent Feature:** FEAT-19 -- Freelancer Branding

## Scope and Non-Goals

**In Scope:**
- Uploading, replacing, and previewing a single logo
- Selecting and previewing a single brand colour, including the inline legibility-adjustment notice
- Resetting both the logo and the brand colour to the clean neutral default in one action
- Dana's read-only rendering of the same screen inside a logged support session (FEAT-31)
- Entry from Onboarding / First-Run Setup (FEAT-20) and from Settings & Account Management (FEAT-21), and an outbound link into Custom Domain per Freelancer (FEAT-27, Later)

**Non-Goals:**
- Validating the logo's file size/format or adjusting the brand colour for legibility -- owned by FEAT-19.SPEC-002 (Branding Upload & Legibility Validation Rules); this screen applies that spec's rules but does not define them
- Applying the saved logo/colour (or the neutral default) to client-facing screens and emails elsewhere in the product -- owned by FEAT-19.SPEC-003 (Branding Application & Fallback Rule); this screen only captures and persists the values
- Supporting more than one logo or more than one brand colour per account -- excluded per product-features.md's Validation & Limits: "one logo and one primary colour per freelancer account in v1"; a richer brand kit is a deliberate later-iteration exclusion, not a limitation of this screen
- Letting Owen, Priya, or any client-side role change branding -- excluded per the Access Matrix (user-persona.md): both client-contact roles carry "None" on Branding, Onboarding & Settings, and scope-boundaries.md SC-01 excludes any second branding editor since the product has no team-of-many account model

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-20 (Onboarding / First-Run Setup) -- guided setup "Set branding" step | Nadia reaches Step 3 of Freelancer First-Time Setup | None -- form starts from whatever Branding Profile exists (typically none yet); the step is presented inline and is skippable |
| FEAT-21 (Settings & Account Management) -- Settings > Branding | Nadia navigates to Settings > Branding, including to finish a previously skipped step | Pre-populated with the current Branding Profile (or the Empty state if nothing was ever set) |
| FEAT-31 (Operator Support Access) -- support session opened | A support session is opened on the freelancer's account and Dana navigates to Branding within it | Read-only render of the freelancer's current Branding Profile; no save or reset control is included |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Upload/replace logo, select/replace brand colour, reset both to default, save | -- |
| Owen (Client Primary Contact) | No | No | Structurally unreachable -- Owen authenticates only into his client portal session (FEAT-05) and the portal exposes no link or URL to this screen at all (Access Matrix: Branding, Onboarding & Settings = None). If a stale or guessed direct link is opened, he is redirected to his own client portal home with no branding-specific message |
| Priya (Client Reviewer Contact) | No | No | Same as Owen: structurally unreachable through the client portal session; a stale or guessed link redirects to her own client portal home |
| Dana (Support Operator) | Full screen, read-only | None -- no upload, colour, save, or reset control is rendered | If Dana attempts a direct action outside an active support session (no session context), the screen is unreachable to her entirely; navigation to Branding only exists from within an open session (FEAT-31) |
| Unauthenticated | No | No | Redirected to the freelancer sign-in screen; after signing in, Nadia lands back on this screen only if she was routed here mid-flow (e.g., mid-onboarding), otherwise she lands on her default dashboard (FEAT-12) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Any unsaved logo selection or colour choice in progress is discarded -- this screen has no offline/draft persistence -- and must be re-entered after re-authenticating |

## Layout and Content

**Header:** Screen title "Branding" with a back arrow (returns to the entry source: FEAT-20's onboarding flow or FEAT-21's Settings area) and, for Nadia only, a "Save" action button (right-aligned). For Dana, the header instead shows a persistent banner: "Viewing in a read-only support session" in place of the Save button.

**Body:** Two primary sections stacked vertically, followed by a live preview:
- **Logo section:** The current logo shown in a preview tile (or a neutral placeholder icon when no logo is set), an "Upload logo" control below it (opens a file picker; relabelled "Replace logo" once a logo exists), and a line of hint text stating the accepted format and size limit drawn from FEAT-19.SPEC-002.
- **Brand colour section:** The current brand colour shown as a swatch with its value (or the neutral default swatch when unset), and a "Choose colour" control below it that opens a colour picker. When FEAT-19.SPEC-002's legibility adjustment has been applied to the active colour, an inline notice sits directly below the swatch.
- **Preview:** A small, non-interactive mock-up of one client-facing surface (a sample proposal header) below both sections, rendering the in-progress logo and colour exactly as FEAT-19.SPEC-003 would apply them, so Nadia can judge the effect before saving.
- **Reset control:** A "Reset to default" text action below the preview, visible to Nadia only, disabled when the Branding Profile already holds no logo and no colour (nothing to reset).

For Dana's read-only render, the same three regions appear (logo, colour, preview) with the Upload, Choose colour, and Reset controls omitted entirely -- she sees values only.

**Onboarding Skip (not a control on this screen):** "Skip" belongs to FEAT-20's step chrome, not to this screen. When this screen is presented inline in FEAT-20's "Set branding" step, FEAT-20 renders its own "Skip" control outside this screen's header and body; this screen defines no Skip element, and the Interactions table therefore has no Skip row. When Nadia taps FEAT-20's Skip, FEAT-20 advances to its next step, any unsaved selection on this screen is discarded without a prompt (skipping is an explicit decision to set nothing), and the neutral default stays in place. Opened from FEAT-21 Settings, no Skip control exists.

**Load-failure banner:** If the current Branding Profile cannot be loaded on open, a banner replaces the Body (see the Load Failed state): "We couldn't load your branding. Try again." with a "Try again" control for Nadia; Dana sees the read-only variant (see Read-only Load Failed).

**Footer:** A "Go further with your own domain" link to FEAT-27 (Custom Domain per Freelancer, Later phase), visible to Nadia only.

### Responsive Behavior

- **Compact breakpoint:** Logo section, colour section, and preview stack in a single column, full width; Save remains in the header.
- **Medium size class and above:** The logo and colour sections sit side by side in two columns, with the preview spanning full width beneath them; no structural change beyond this two-column regrouping.
- **Preview mock-up:** Scales proportionally with available width at every size; it never becomes interactive or gains its own controls.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap, no unsaved changes (no logo or colour selection differs from the saved Branding Profile) | Navigate to the entry source (FEAT-20's onboarding flow or FEAT-21 Settings & Account Management) | Screen closes | Standard transition back |
| Back arrow | Tap, with unsaved changes | Show the unsaved-changes dialog | Dialog appears; screen stays open behind it | Dialog text: "You have unsaved branding changes. Discard?" with "Discard" and "Keep Editing"; focus moves to "Keep Editing" |
| Unsaved-changes dialog: "Discard" | Tap | Drop all unsaved logo and colour selections and navigate to the entry source | Pending selections cleared; screen closes | Standard transition back; saved branding unchanged |
| Unsaved-changes dialog: "Keep Editing" | Tap (or Escape) | Close the dialog only | Dialog closes; unsaved selections and all pending values preserved | Focus returns to the Back arrow |
| Error banner "Dismiss" control | Tap | Remove the banner; the previously saved logo and colour stay shown and applied | Error state exits to Populated or Empty (whichever matches the saved profile); any pending selection that did not fail is retained | Banner disappears; focus returns to the control that caused the error |
| Load-failure "Try again" control | Tap (Nadia only) | Re-request the Branding Profile | Screen enters Loading; on success Populated or Empty, on failure Load Failed again | Loading indicator; on repeated failure the same banner reappears |
| Upload/Replace logo control | Tap, select a file | Validate the file via FEAT-19.SPEC-002 | Preview tile shows the Uploading state, then the new logo thumbnail on success | Success: thumbnail updates and the live preview reflects it. Failure: inline error per FEAT-19.SPEC-002's message; previous logo (if any) stays shown and stays the applied one until a new upload succeeds |
| Choose colour control | Tap, select a colour | Validate/adjust the colour via FEAT-19.SPEC-002 | Swatch updates to the chosen (or adjusted) colour; live preview updates | If the colour required legibility adjustment, the inline notice "Your colour was adjusted slightly so text stays readable against it." appears beneath the swatch |
| Reset to default | Tap (Nadia only, enabled only when a logo or colour is currently set) | Show a confirmation dialog | Dialog appears | Dialog text: "Reset your branding to the plain default? Your logo and colour will be removed from every client-facing screen and email." with "Reset" and "Cancel" |
| Reset confirmation dialog: "Cancel" | Tap (or Escape) | Close the dialog only; nothing is cleared or saved | Dialog closes; logo, colour, and any unsaved pending selections unchanged | Focus returns to the Reset to default control |
| Reset confirmation dialog | Tap "Reset" | Clear both logo and colour to unset and save immediately | Screen returns to the Empty appearance; preview shows the neutral default | Toast: "Branding reset to default." |
| Save button (nothing changed) | Disabled (Nadia only) whenever no logo or colour selection differs from the saved Branding Profile, including immediately after opening; it becomes enabled on the first differing selection | Cannot be triggered | Button appears dimmed and unfocusable-for-action | Announced as "Save, unavailable: no changes" to assistive technology |
| Save button | Tap (Nadia only, enabled) | 1. Confirm all pending values already passed FEAT-19.SPEC-002 validation. 2. Persist the Branding Profile. 3. Trigger FEAT-19.SPEC-003 to apply the new values everywhere. | Button shows a loading state during save | Success: toast "Branding saved." Failure: inline error banner; previously saved values remain applied unchanged |
| Save button (while saving) | Tap | No action -- ignored while a save is already in progress | Button remains in its loading state | No additional feedback |
| "Go further with your own domain" link | Tap (Nadia only) | Navigate to FEAT-27 (Custom Domain per Freelancer) | Screen closes | Standard transition |
| Preview mock-up | -- | Display-only -- reflects in-progress selections; not interactive | None | None |

### Accessibility Notes

- **Focus order:** Back arrow -> Upload/Replace logo control -> Choose colour control -> Reset to default (when enabled) -> preview mock-up (announced as a non-interactive illustrative region, skippable) -> Save.
- **Legibility notice:** When the inline "Your colour was adjusted slightly..." notice appears, it is announced to assistive technology immediately and is programmatically associated with the colour swatch.
- **Upload feedback:** Both the Uploading state and any resulting error are announced; on error, focus moves to the logo control so the retry path is immediate.
- **Save feedback:** The "Branding saved." toast and any save-failure banner are both announced on appearance.
- **Read-only banner (Dana):** The "Viewing in a read-only support session" banner is announced once when the screen loads, so Dana's assistive-technology experience makes the missing controls explicable rather than confusing.
- **Keyboard alternatives:** Every action on this screen -- including the colour picker -- is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | Skeleton placeholders for the logo tile, swatch, and preview; all controls (including Save) disabled; Dana sees the same skeleton with the read-only banner | Screen opens and the Branding Profile is being read | Profile read succeeds (Empty or Populated) or fails (Load Failed / Read-only Load Failed) |
| Load Failed | Body replaced by the banner "We couldn't load your branding. Try again." with a "Try again" control; Upload, Choose colour, Reset, and Save are hidden so Nadia cannot save over a profile she could not see | The Branding Profile read fails (network error or server error) for Nadia | Nadia taps "Try again" (returns to Loading) or navigates back |
| Read-only Load Failed (Dana) | Read-only banner stays; body shows "Branding could not be loaded for this account." with no retry or edit control; Dana can only leave the screen or re-open it | The Branding Profile read fails during a support session | Dana leaves and re-opens the screen, or the support session closes |
| Empty (default) | Neutral placeholder logo icon, default colour swatch, Reset control disabled, preview shows the clean neutral default | No Branding Profile exists yet, or one exists with both fields unset | Nadia uploads a logo or selects a colour |
| Populated | Current logo thumbnail and current colour swatch shown, Reset control enabled, preview reflects both | Branding Profile has a logo and/or a colour set | Nadia changes a value, resets, or leaves the screen |
| Uploading | Logo tile shows a progress indicator in place of the thumbnail | A logo file is selected and passes initial format/size checks | Upload completes (success shows the new thumbnail) or fails (Error state) |
| Legibility Adjusted | Inline notice shown beneath the colour swatch; the swatch itself already shows the adjusted colour | FEAT-19.SPEC-002 adjusts the selected colour for legibility | Nadia selects a different colour, or navigates away/saves (notice persists until the next colour change) |
| Saving | Save button shows a loading spinner, all controls disabled | Nadia taps Save | Save completes (success) or fails (Error state) |
| Error | Inline error banner at the top of the affected section, with the exact validation message from FEAT-19.SPEC-002 or a generic save-failure message; previous values remain shown as active | A logo upload fails validation, a network interruption occurs mid-upload, or a save fails | Nadia corrects the input and retries successfully, or taps the banner's "Dismiss" control (see Interactions) and keeps the previous branding |
| Read-only (Dana) | Same logo/colour/preview regions with no Upload, Choose colour, Save, or Reset controls; persistent "Viewing in a read-only support session" banner in the header | Dana opens this screen from within an active support session (FEAT-31) | The support session closes |
| Offline/Degraded | N/A -- branding is set from the freelancer's desktop with expected connectivity (product-features.md, States field); this screen has no offline-queued behavior | -- | -- |

## Validation Rules

Validation governed by FEAT-19.SPEC-002 (Branding Upload & Legibility Validation Rules). See that spec for the logo file size/format rules, the one-logo/one-colour-per-account limit, and the colour legibility adjustment. This screen applies those rules on file selection (logo) and on colour selection, and re-confirms them on Save.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap (entered from onboarding) | FEAT-20's onboarding flow, at the branding step | FEAT-20 (Onboarding / First-Run Setup) |
| Back arrow tap (entered from Settings) | Settings & Account Management area | FEAT-21 (Settings & Account Management) |
| Successful save (entered from onboarding) | FEAT-20's onboarding flow, which proceeds to its next step | FEAT-20 (Onboarding / First-Run Setup) |
| Successful save (entered from Settings) | Settings & Account Management area | FEAT-21 (Settings & Account Management) |
| Unsaved-changes dialog "Discard" (from Back arrow) | The entry source (FEAT-20's onboarding flow or FEAT-21 Settings) | FEAT-20 or FEAT-21, per entry |
| FEAT-20's own "Skip" control (owned by FEAT-20, not rendered by this screen) | FEAT-20's onboarding flow continues, leaving the neutral default in place | FEAT-20 (Onboarding / First-Run Setup) |
| "Go further with your own domain" link tap | Custom domain setup | FEAT-27 (Custom Domain per Freelancer, Later phase) |
| Reset confirmed | Remains on this screen, now showing the Empty appearance | -- |

## Data Model

**Creates:** Branding Profile -- created the first time Nadia saves a logo and/or colour (whether reached directly, via FEAT-20's guided step, or via FEAT-21). Before this first save, the profile is effectively absent and the screen shows its Empty state.
**Reads:** Branding Profile -- logo and brand_colour fields, loaded when the screen opens (or the Empty state when neither exists). Also reads the signed-in Freelancer Account to scope the profile to the correct owner.
**Updates:** Branding Profile -- logo and/or brand_colour fields on Save; Reset clears both fields back to unset. A save from a second concurrent session simply replaces the first (dependency map, Contention: "last-write-wins").
**Deletes:** None -- this screen never deletes the Branding Profile record; the record is deleted only by FEAT-24 (Account Deletion) when the freelancer's account itself is deleted.

## Business Rules

- Field-level validation and the legibility adjustment are governed entirely by FEAT-19.SPEC-002; this screen applies that spec's rules directly and shows its exact messages -- it defines no validation of its own.
- XBR-31: once a save succeeds (or a reset is confirmed), FEAT-19.SPEC-003 applies the resulting logo/colour (or the neutral default) to every client-facing screen and email immediately.
- Dana's read-only rendering is governed by XBR-29 (support sessions are read-only in every feature) and the Access Matrix's Support Access entitlement (ASMP-18) -- no save or reset control is ever included in her view, regardless of what the freelancer's Branding Profile holds.
- Exactly one Branding Profile exists per Freelancer Account (Feature Dependency Map); there is no independent "create" action distinct from the first save, and no way to hold more than one logo or colour at a time.
- A failed logo upload never changes what is applied elsewhere -- the previously saved logo and colour remain active on every client-facing surface until a new upload succeeds (product-features.md, States field: Error).

## Edge Cases

- **Nadia has this screen open in two browser sessions and saves different values in each** -- The second save simply replaces the first; the first session's view is not corrected in place and shows the stale values until it is reloaded. Resolution: last-write-wins, per the dependency map's Contention note for the Branding Profile.
- **A logo upload fails partway through (bad file, size/format rejection, or a network interruption)** -- The previous logo and colour remain active and applied everywhere; an inline error explains the failure per FEAT-19.SPEC-002's exact message, and nothing changes until a new upload succeeds.
- **Nadia taps Save twice in rapid succession** -- The second tap is ignored while the first save is in progress (button remains in its loading state).
- **Nadia navigates away with an unsaved logo or colour selection** -- Confirmation dialog: "You have unsaved branding changes. Discard?" with "Discard" and "Keep Editing" options, as defined in the Interactions table.
- **The Branding Profile cannot be loaded on open** -- Nadia sees the Load Failed state and can retry; Dana sees the Read-only Load Failed state. No save is possible until a load succeeds.
- **Nadia taps Reset when nothing has ever been set** -- The Reset control is disabled in this state, so the action cannot be triggered; there is nothing to confirm.
- **Nadia resets her branding immediately after uploading a logo, before the earlier save has finished propagating** -- The reset (the more recent action) takes precedence; FEAT-19.SPEC-003 applies the neutral default to any client-facing render from that point forward.
- **The colour Nadia selects sits exactly at the legibility threshold defined by FEAT-19.SPEC-002** -- It passes unadjusted; no inline notice appears (the threshold is inclusive, per FEAT-19.SPEC-002).
- **Dana's support session closes while she is viewing this screen** -- She is redirected out of the freelancer's account view entirely, consistent with every other screen's behavior under FEAT-31; there is no separate branding-specific closing behavior.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-19.SPEC-002 (Branding Upload & Legibility Validation Rules) | References (outbound) | Logo file size/format validation, the one-logo/one-colour-per-account limit, and colour legibility adjustment are applied directly from this spec |
| FEAT-19.SPEC-003 (Branding Application & Fallback Rule) | Triggers (outbound) | A successful save or a confirmed reset triggers immediate application of the resulting branding (or the neutral default) everywhere |
| FEAT-20 (Onboarding / First-Run Setup) | Navigation (inbound) | Guided setup's skippable "Set branding" step enters here |
| FEAT-21 (Settings & Account Management) | Navigation (inbound) | Nadia returns here to set or change branding from Settings |
| FEAT-27 (Custom Domain per Freelancer) | Navigation (outbound) | "Go further with your own domain" link (Later phase) |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only entry point during a logged support session |
| FEAT-24 (Data Export & Account Deletion) | References (informational) | The Branding Profile this screen manages is deleted only as part of account deletion, never by this screen |

## Analytics and Success Signals

No success-metrics.md metric names Freelancer Branding (FEAT-19) as its Connected Feature (checked against every metric in success-metrics.md); every event below is therefore recorded for product usage visibility only, and this gap is intentional rather than dropped.

| Event | Properties | Emitted When | Supports Metric |
|-------|-------------|----------------|--------------------|
| branding_logo_uploaded | entry source (onboarding / settings), whether this replaces an existing logo | A logo upload passes FEAT-19.SPEC-002 validation and the profile is saved | N/A -- no success-metrics.md metric is connected to Freelancer Branding (FEAT-19); recorded for product usage visibility only |
| branding_color_set | entry source (onboarding / settings), whether the colour required legibility adjustment | A brand colour is saved, adjusted or not | N/A -- no success-metrics.md metric is connected to Freelancer Branding (FEAT-19); recorded for product usage visibility only |
| branding_reset_to_default | entry source (onboarding / settings), whether a logo, a colour, or both were cleared | Nadia confirms Reset to default | N/A -- no success-metrics.md metric is connected to Freelancer Branding (FEAT-19); recorded for product usage visibility only |

## Acceptance Criteria

**FEAT-19.SPEC-001-AC-01:** Given Nadia is on the Branding Settings screen with no branding ever set, when she uploads a logo that passes FEAT-19.SPEC-002's checks and taps Save, then the logo is saved, a "Branding saved." toast appears, and FEAT-19.SPEC-003 applies it to client-facing screens immediately.

**FEAT-19.SPEC-001-AC-02:** Given Nadia uploads a file that fails FEAT-19.SPEC-002's format or size rule, when the check runs, then the inline error from that spec appears, the upload does not complete, and any previously saved logo remains active and applied.

**FEAT-19.SPEC-001-AC-03:** Given Nadia selects a colour that would make text hard to read, when she confirms the selection, then the swatch shows the colour FEAT-19.SPEC-002 adjusted it to and the inline notice "Your colour was adjusted slightly so text stays readable against it." appears.

**FEAT-19.SPEC-001-AC-04:** Given Nadia already has a logo and colour set, when she taps Reset to default and confirms, then both fields clear to unset, the screen shows the Empty appearance, and a "Branding reset to default." toast appears.

**FEAT-19.SPEC-001-AC-05:** Given Nadia has never set any branding, when she looks at the Reset to default control, then it is disabled and cannot be triggered.

**FEAT-19.SPEC-001-AC-06:** Given Nadia has unsaved logo or colour changes, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved branding changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-19.SPEC-001-AC-07:** Given Nadia taps Save and the request is still in progress, when she taps Save again, then the second tap is ignored and the button remains in its loading state.

**FEAT-19.SPEC-001-AC-08:** Given Nadia's logo upload is interrupted by a network failure, when the interruption occurs, then an inline error banner appears and her previous logo and colour remain active and applied on every client-facing surface.

**FEAT-19.SPEC-001-AC-09:** Given Dana opens Branding Settings inside an active support session, when the screen renders, then she sees the freelancer's current logo and colour with no Upload, Choose colour, Save, or Reset control, and the "Viewing in a read-only support session" banner is shown.

**FEAT-19.SPEC-001-AC-10:** Given Owen or Priya attempts to reach Branding Settings, when the attempt is made, then no such screen or link exists anywhere in their client portal session, and a stale or guessed direct link redirects them to their own client portal home.

**FEAT-19.SPEC-001-AC-11:** Given an unauthenticated visitor opens the Branding Settings URL directly, when the request is made, then they are redirected to the freelancer sign-in screen.

**FEAT-19.SPEC-001-AC-12:** Given Nadia's session expires while she has unsaved changes on this screen, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and the unsaved logo or colour selection is discarded.

**FEAT-19.SPEC-001-AC-13:** Given Nadia has this screen open in two sessions and saves different colours in each, when both saves complete, then the second save's colour is the one applied everywhere, per last-write-wins.

**FEAT-19.SPEC-001-AC-14:** Given Nadia reaches this screen from FEAT-20's onboarding "Set branding" step, when she taps the "Skip" control that FEAT-20's step chrome provides (this screen renders none) instead of setting anything, then any unsaved selection is discarded without a prompt and the onboarding flow continues with the neutral default left in place and no progress is lost.

**FEAT-19.SPEC-001-AC-15:** Given Nadia reaches this screen from FEAT-21 Settings to finish a previously skipped branding step, when the screen loads, then it is pre-populated with whatever was set before (or remains empty if nothing was).

**FEAT-19.SPEC-001-AC-16:** Given Nadia is on the Branding Settings screen, when she taps "Go further with your own domain", then she is navigated to FEAT-27's custom domain setup.

**FEAT-19.SPEC-001-AC-17:** Given Nadia selects a colour exactly at FEAT-19.SPEC-002's legibility threshold, when she confirms it, then it is accepted unadjusted and no inline notice appears.

**FEAT-19.SPEC-001-AC-18:** Given Nadia has unsaved changes and the unsaved-changes dialog is open, when she taps "Keep Editing", then the dialog closes, she stays on the screen, and her pending logo and colour selections are unchanged; and when she instead taps "Discard", then the selections are dropped and she returns to the entry source with the saved branding unchanged.

**FEAT-19.SPEC-001-AC-19:** Given an error banner is shown after a failed upload or save, when Nadia taps "Dismiss", then the banner disappears and her previously saved logo and colour remain shown and applied.

**FEAT-19.SPEC-001-AC-20:** Given Nadia has the Reset confirmation dialog open with a logo and colour set, when she taps "Cancel", then the dialog closes and her logo, colour, and any unsaved selections are unchanged.

**FEAT-19.SPEC-001-AC-21:** Given Nadia has just opened the screen and changed nothing, when she looks at the Save button, then it is disabled; and when she selects a new colour, then Save becomes enabled.

**FEAT-19.SPEC-001-AC-22:** Given the Branding Profile cannot be loaded when Nadia opens the screen, when the read fails, then she sees "We couldn't load your branding. Try again." with a "Try again" control, no Save, Upload, Choose colour, or Reset control is shown, and tapping "Try again" re-requests the profile.

**FEAT-19.SPEC-001-AC-23:** Given the Branding Profile cannot be loaded while Dana views the screen in a support session, when the read fails, then she sees the read-only banner and "Branding could not be loaded for this account." with no retry or edit control.

**FEAT-19.SPEC-001-AC-24:** Given Nadia or Dana opens the screen, when the Branding Profile read is in progress, then skeleton placeholders are shown and every control is disabled until the read succeeds or fails.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 16 | 16 |
| States | 11 (loading, load failed, read-only load failed, empty, populated, uploading, legibility adjusted, saving, error, read-only, offline/degraded N/A) | 11 |
| Business Rules | 5 | 5 |
| Edge Cases | 9 | 9 |



# Logic/Rule Spec: Branding Upload & Legibility Validation Rules

## Overview

**Name:** Branding Upload & Legibility Validation Rules
**ID:** FEAT-19.SPEC-002
**Type:** Logic/Rule
**Purpose:** Governs the logo file size/format limits, the one-logo/one-colour-per-account limit, and the automatic legibility adjustment applied to a brand colour that would make text hard to read.
**Parent Feature:** FEAT-19 -- Freelancer Branding
**Governed Entity:** Branding Profile

## Scope and Non-Goals

**In Scope:**
- The logo file's format and size-ceiling rules
- The one-logo-per-account and one-colour-per-account cardinality rules
- The colour legibility adjustment: detection and the corrected-value derivation
- Authorization for who may set, change, or reset each field, and who may only view the result

**Non-Goals:**
- Capturing the upload or the colour selection itself, or showing the resulting error/notice on screen -- owned by FEAT-19.SPEC-001 (Branding Settings), which calls this spec's rules and displays their exact messages
- Applying the validated logo/colour (or the neutral default) to client-facing screens and emails -- owned by FEAT-19.SPEC-003 (Branding Application & Fallback Rule), which only ever consumes values that have already passed this spec; it never re-validates them
- Checking the logo or colour for trademark, brand-guideline, or content-appropriateness concerns -- excluded per the Brief's Non-Goals: the product definition's only stated checks are technical (file size/format) and accessibility-driven (legibility); the Branding Profile's Data Sensitivity is "None," so no compliance-review capability applies here

## Governed Entity

**Entity:** Branding Profile
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| logo | image file (optional) | The freelancer's uploaded logo image; unset means the neutral default logo treatment applies |
| brand_colour | colour value (optional) | The freelancer's single primary colour; unset means the neutral default palette applies |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-------------|------------------------|
| FEAT-19.SPEC-001 | Branding Settings | Logo format/size check on file selection, before the upload proceeds; colour legibility check on colour selection and re-confirmed on Save; cardinality rules and authorization applied on screen entry and on Save |
| FEAT-19.SPEC-003 | Branding Application & Fallback Rule | Consumes only values that have already passed this spec's checks; performs no validation of its own |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|-------------------------------|----------------|---------------------|-----------|
| logo | Optional; when provided, must be a standard image format (JPEG, PNG, or SVG) | When a file is provided | On file selection | "Please upload a logo in JPEG, PNG, or SVG format." | Yes |
| logo | When provided, file size must not exceed platform parameter: `branding-logo-file-size-ceiling` | When a file is provided | On file selection, before the upload proceeds | "This logo is larger than the size limit. Choose a smaller file and try again." | Yes |
| logo | Exactly one logo per Freelancer Account -- a successful new upload replaces the existing logo entirely; there is no multi-logo list | Always | On save | N/A -- not an error; a successful save silently supersedes the prior logo | No |
| brand_colour | Optional; when provided, must be a valid colour value | When a colour is provided | On selection | "Please choose a valid colour." | Yes |
| brand_colour | A colour whose contrast against the standard text placed over it falls below platform parameter: `branding-color-legibility-contrast-ratio` is automatically adjusted to the nearest legible shade of the same hue before it is stored (see Defaults and Derivations) | When the selected colour's contrast is below the threshold | On selection, re-confirmed on save | Non-blocking inline notice (shown by FEAT-19.SPEC-001): "Your colour was adjusted slightly so text stays readable against it." | No -- adjusts the value rather than rejecting it |
| brand_colour | Exactly one brand colour per Freelancer Account -- a successful new selection replaces the existing colour entirely; there is no multi-colour list | Always | On save | N/A -- not an error; a successful save silently supersedes the prior colour | No |

## Cross-Field Rules

No cross-field rule applies to the Branding Profile: logo and brand_colour are independent optional values, each validated, adjusted (colour only), and superseded entirely on its own -- setting one has no effect on the validity or presence of the other. Each field's own fallback to the neutral default (when unset) is likewise independent and is defined by FEAT-19.SPEC-003, not by an interaction between the two fields.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|--------------|-------------|---------------------------------------------------|
| Upload/replace logo | Nadia (Freelancer) | Always, subject to the Field Validation Rules above | -- |
| Upload/replace logo | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | No upload control exists anywhere in the client portal; the Access Matrix's Branding, Onboarding & Settings entitlement for both roles is None |
| Upload/replace logo | Dana (Support Operator) | Never | No upload control is rendered in a support session (FEAT-19.SPEC-001's read-only render); Dana's Branding, Onboarding & Settings entitlement is View only (XBR-29) |
| Set/replace brand colour | Nadia (Freelancer) | Always, subject to the Field Validation Rules above | -- |
| Set/replace brand colour | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | No colour control exists anywhere in the client portal; Access Matrix entitlement is None |
| Set/replace brand colour | Dana (Support Operator) | Never | No colour control is rendered in a support session; Dana's entitlement is View only (XBR-29) |
| Reset to default (clear both fields) | Nadia (Freelancer) | Always, only enabled when at least one field currently holds a value | -- |
| Reset to default (clear both fields) | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | No reset control exists anywhere in the client portal; Access Matrix entitlement is None |
| Reset to default (clear both fields) | Dana (Support Operator) | Never | No reset control is rendered in a support session; Dana's entitlement is View only (XBR-29) |
| View current logo and colour | Nadia (Freelancer) | Always | -- |
| View current logo and colour | Dana (Support Operator) | Always, read-only, only during an active support session on that freelancer's account (FEAT-31) | Outside an active support session, the Branding Settings screen is unreachable to Dana; no separate denied experience since no navigation path exists |
| View current logo and colour | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never (this rule governs the settings screen itself; both roles do see the resulting applied branding elsewhere, governed by FEAT-19.SPEC-003) | No Branding Settings screen or link exists in the client portal; Access Matrix entitlement is None |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------------|-----------------|---------------------------|
| logo | No default value -- remains unset until Nadia explicitly uploads one; the neutral default logo treatment used elsewhere is a rendering fallback owned by FEAT-19.SPEC-003, not a stored value here | On account creation, and after every reset | Yes -- by uploading a logo at any time |
| brand_colour (stored value) | When the raw colour Nadia selects has a contrast ratio below platform parameter: `branding-color-legibility-contrast-ratio` against the standard text placed over it, the stored value is the nearest colour along the same hue whose contrast ratio meets or exceeds that threshold. If no shade of that hue clears the threshold, the stored value is the darkest legible shade of that hue (or, for a hue with no legible shade at all, the nearest neutral dark shade), so a value that meets the threshold is always stored | On selection, and re-confirmed on save | Yes -- Nadia can pick a different original colour to obtain a different adjusted result, but cannot force a colour that fails the threshold to be stored unadjusted |
| brand_colour (default) | No default value -- remains unset until Nadia explicitly selects one; the neutral default palette used elsewhere is a rendering fallback owned by FEAT-19.SPEC-003 | On account creation, and after every reset | Yes -- by selecting a colour at any time |

## Business Rules

- Exactly one Branding Profile exists per Freelancer Account (Feature Dependency Map); the one-logo and one-colour limits above are simply this cardinality applied to each field -- there is no versioning or history of prior logos/colours retained.
- FEAT-19.SPEC-003 only ever applies logo and brand_colour values that have already passed this spec's checks (including the legibility adjustment) -- it performs no independent validation, per the Brief's Shared Validation note.
- The legibility adjustment applies to brand_colour only; the logo's own internal contrast or readability is not evaluated by this feature -- ASMP-27's screen-reader/keyboard/no-colour-alone baseline is a platform-wide accessibility rule that this spec's colour adjustment supports but does not by itself fully satisfy.
- Resetting to default clears both fields back to unset in a single action; it is an update to the existing Branding Profile record, never a deletion of the record itself (the record is deleted only by FEAT-24, per the dependency map).

## Edge Cases

- **Logo file exactly at platform parameter: `branding-logo-file-size-ceiling`** -- Passes validation; one byte over fails.
- **Logo file in an unsupported format but with a renamed extension matching an accepted one** -- The format check inspects the actual file content, not the filename, so a mismatched or corrupted file still fails with "Please upload a logo in JPEG, PNG, or SVG format."
- **Brand colour with contrast ratio exactly at platform parameter: `branding-color-legibility-contrast-ratio`** -- Passes unadjusted; the boundary is inclusive.
- **Brand colour whose entire hue never clears the legibility threshold (e.g., a very light, low-contrast hue)** -- Adjusted to the darkest legible shade of that hue per Defaults and Derivations, ensuring a legible value is always stored even at this extreme.
- **Nadia uploads a new logo before the previous upload attempt's error has been dismissed** -- The new attempt is evaluated independently; a successful new upload supersedes any prior error state.
- **Nadia selects a new colour immediately after a legibility adjustment was applied to the previous selection** -- The new colour is evaluated fresh against the threshold; the prior adjustment has no bearing on it.
- **Reset attempted when the Branding Profile already holds no logo and no colour** -- FEAT-19.SPEC-001 disables the Reset control in this state, so this spec's clearing logic is never invoked with nothing to clear.
- **Dana's support session is opened for a freelancer whose current colour was legibility-adjusted** -- Dana sees the stored (already-adjusted) colour value read-only; the adjustment notice itself is a save-time UI event owned by FEAT-19.SPEC-001 and is not shown retroactively to a later viewer.

## Acceptance Criteria

**FEAT-19.SPEC-002-AC-01:** Given Nadia selects a logo file in an accepted format, when the format check runs, then it passes.

**FEAT-19.SPEC-002-AC-02:** Given Nadia selects a logo file in an unsupported format, when the format check runs, then she sees "Please upload a logo in JPEG, PNG, or SVG format." and the upload does not proceed.

**FEAT-19.SPEC-002-AC-03:** Given Nadia selects a logo file exactly at platform parameter: `branding-logo-file-size-ceiling`, when the size check runs, then the file passes.

**FEAT-19.SPEC-002-AC-04:** Given Nadia selects a logo file one byte over platform parameter: `branding-logo-file-size-ceiling`, when the size check runs, then she sees "This logo is larger than the size limit. Choose a smaller file and try again." and the upload does not proceed.

**FEAT-19.SPEC-002-AC-05:** Given Nadia already has a logo saved, when she uploads and saves a new one that passes validation, then the new logo replaces the old one entirely and no second logo is retained.

**FEAT-19.SPEC-002-AC-06:** Given Nadia selects a colour with contrast at or above platform parameter: `branding-color-legibility-contrast-ratio`, when she confirms it, then it is stored unadjusted and no notice is shown.

**FEAT-19.SPEC-002-AC-07:** Given Nadia selects a colour with contrast below platform parameter: `branding-color-legibility-contrast-ratio`, when she confirms it, then the stored value is the nearest legible shade of the same hue, and FEAT-19.SPEC-001 shows "Your colour was adjusted slightly so text stays readable against it."

**FEAT-19.SPEC-002-AC-08:** Given Nadia selects a colour whose entire hue never clears the legibility threshold, when the adjustment runs, then the darkest legible shade of that hue is stored, never an illegible value.

**FEAT-19.SPEC-002-AC-09:** Given Nadia already has a colour saved, when she selects and saves a new one, then the new (possibly adjusted) colour replaces the old one entirely.

**FEAT-19.SPEC-002-AC-10:** Given Nadia attempts to upload a logo, set a colour, or reset to default, when the action is evaluated, then it is allowed subject to the validation rules above; given Owen, Priya, or Dana attempts any of these same three actions, then no control for it exists anywhere they can reach.

**FEAT-19.SPEC-002-AC-11:** Given Dana opens a support session on a freelancer's account, when she views Branding, then she sees the current logo and colour read-only; given Owen or Priya looks for the Branding Settings screen, then none exists in the client portal.

**FEAT-19.SPEC-002-AC-12:** Given Nadia has never set a logo or a colour, when FEAT-19.SPEC-001 renders the Reset control, then it is disabled, since this spec's clearing logic has nothing to act on.

**FEAT-19.SPEC-002-AC-13:** Given Nadia's Branding Profile has an already-legibility-adjusted colour, when FEAT-19.SPEC-003 reads it to apply it elsewhere, then it applies that stored value directly, performing no re-validation or re-adjustment of its own.

**FEAT-19.SPEC-002-AC-14:** Given Nadia confirms Reset to default, when the reset is applied, then both logo and brand_colour are cleared to unset on the existing Branding Profile record, and the record itself is not deleted.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 0 (none apply -- documented above) | 0 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Branding Application & Fallback Rule

## Overview

**Name:** Branding Application & Fallback Rule
**ID:** FEAT-19.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs how the freelancer's logo and colour -- or the clean neutral default when unset -- are applied to every client-facing screen and email, and how the referral mark coexists with them without overriding them.
**Parent Feature:** FEAT-19 -- Freelancer Branding
**Governed Entity:** Branding Profile

## Scope and Non-Goals

**In Scope:**
- Reading the current Branding Profile and resolving what logo and colour treatment every client-facing screen and email must render, at the moment each one renders or sends
- The clean neutral default that applies when the profile is unset or has been reset
- How the referral mark (FEAT-33, XBR-32) coexists with the applied branding without overriding it
- Serving as the single authoritative home for XBR-31, read by every consuming feature (FEAT-02, FEAT-05, FEAT-06, FEAT-09, FEAT-14, FEAT-27, FEAT-31, FEAT-33)

**Non-Goals:**
- Capturing or validating the logo file, the colour value, the size/format limits, or the legibility adjustment -- owned entirely by FEAT-19.SPEC-002 (Branding Upload & Legibility Validation Rules); this spec only ever reads values that spec has already validated and, where needed, adjusted, and it never re-validates or re-adjusts them
- The Branding Settings screen's own interactions, states, and save/reset behavior -- owned by FEAT-19.SPEC-001 (Branding Settings); this spec governs what happens after a value is already saved, not how it was captured
- Deciding the referral mark's own placement rules, content, or opt-out behavior -- owned by FEAT-33 (Portal Referral Attribution); this spec governs only that the mark and the applied branding coexist without either overriding the other, not the mark's own definition

## Governed Entity

**Entity:** Branding Profile
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| logo | image file (optional) | Already validated by FEAT-19.SPEC-002; this spec reads it as-is -- no validation of its own |
| brand_colour | colour value (optional) | Already validated and, where needed, legibility-adjusted by FEAT-19.SPEC-002; this spec reads it as-is -- no validation of its own |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-------------|------------------------|
| FEAT-02 (Proposal Creation & Sending) | Proposal screens | Applies the resolved logo/colour or the neutral default when rendering any proposal screen a client contact can view |
| FEAT-05 (Client Portal Access) | Every client portal screen | Applies the resolved logo/colour or the neutral default on every screen inside the client portal |
| FEAT-06 (Deliverable Upload & Sharing) | Deliverable review screens | Applies the resolved logo/colour or the neutral default when a deliverable is viewed by a client contact |
| FEAT-09 (Invoice Generation & Sending) | Invoice screens | Applies the resolved logo/colour or the neutral default when an invoice is viewed or paid |
| FEAT-14 (Notifications -- Email) | Every client-facing email | Applies the resolved logo/colour or the neutral default to every email a client contact receives |
| FEAT-27 (Custom Domain per Freelancer, Later) | Custom-domain-served portal pages | Applies the same resolved branding regardless of which address (shared default or custom domain) serves the page |
| FEAT-31 (Operator Support Access) | Branding Settings, read-only view | Reads the same resolved values to render Dana's read-only view (via FEAT-19.SPEC-001) |
| FEAT-33 (Portal Referral Attribution) | Every client-facing page and email | Places the referral mark alongside whatever this spec resolves, per the Coexistence Rule below |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|-------------------------------|----------------|---------------------|-----------|
| logo | No validation beyond data type -- already validated by FEAT-19.SPEC-002 before this spec ever reads it | Always | -- | -- | No |
| brand_colour | No validation beyond data type -- already validated and legibility-adjusted by FEAT-19.SPEC-002 before this spec ever reads it | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|-------------------|-------|---------------------|
| Independent per-field fallback | logo, brand_colour | Each field falls back to the clean neutral default independently of the other: an unset logo renders the neutral default logo treatment regardless of whether a colour is set, and an unset colour renders the neutral default palette regardless of whether a logo is set. A freelancer who has set only one of the two fields sees a mix of her own value and the default for the other | N/A -- descriptive rendering behavior, not a validation failure |
| Referral mark coexistence (XBR-32) | logo, brand_colour (as applied, set or default) | The referral mark renders alongside whatever this spec resolves for the surface -- it never adopts the freelancer's brand colour itself, never replaces the logo, and never causes the applied branding to be hidden or resized | N/A -- descriptive rendering behavior, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|--------------|-------------|---------------------------------------------------|
| View a client-facing screen or email with the resolved branding applied | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Always, wherever each role is otherwise entitled to view the underlying screen or email (this spec adds no additional restriction beyond the consuming spec's own access rules) | -- |
| Change what is applied (edit the underlying logo or colour) | Nadia (Freelancer) | Always, but only through FEAT-19.SPEC-001/FEAT-19.SPEC-002 -- this spec exposes no direct editing surface of its own | -- |
| Change what is applied (edit the underlying logo or colour) | Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Never | No editing surface exists in this spec or anywhere this spec is consumed; the only editing surface anywhere in the product is FEAT-19.SPEC-001, and FEAT-19.SPEC-002's Authorization Rules govern denial for these roles there |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------------|-----------------|---------------------------|
| applied_logo (derived, not stored) | If Branding Profile.logo is set, the applied logo is that file; otherwise the applied logo is the clean neutral default logo treatment (no mark, a plain typographic header per BRIEF.md's Constraints: "clean and professional with lots of white space") | Evaluated fresh on every client-facing render or email composition | No -- only changing the underlying source field (via FEAT-19.SPEC-001) changes what is resolved |
| applied_colour (derived, not stored) | If Branding Profile.brand_colour is set, the applied colour is that (already legibility-adjusted) value; otherwise the applied colour is the neutral default palette's primary colour | Evaluated fresh on every client-facing render or email composition | No -- only changing the underlying source field (via FEAT-19.SPEC-001) changes what is resolved |
| Freshness of the resolved branding | Always reflects the current Branding Profile at the moment of render or send -- there is no caching that would show a stale value once a save or reset has completed | Always | No |

## Business Rules

- XBR-31: the freelancer's logo and brand colour apply to every client-facing screen and email, fall back to a clean neutral default, and are adjusted for legibility (by FEAT-19.SPEC-002, before this spec ever sees the value); this spec is the sole authority for that rule, read by FEAT-02, FEAT-05, FEAT-06, FEAT-09, FEAT-14, FEAT-27, FEAT-31, and FEAT-33.
- A save or a confirmed reset on FEAT-19.SPEC-001 applies the resulting values (or the default) immediately -- every client-facing screen and email that renders or sends after that point uses the new resolution; there is no separate propagation delay or rollout step.
- The single Branding Profile is scoped to exactly one Freelancer Account; no per-client, per-project, or per-email variation exists -- every client of a given freelancer sees the same applied branding.
- XBR-32: the referral mark (FEAT-33) sits alongside the applied branding on every client-facing page and email on every plan in MVP; it is never suppressed by branding being set, and it never causes the applied logo or colour to be hidden, resized, or overridden.
- The shared, unbranded default address (FEAT-27, Later) always remains available regardless of what this spec resolves for logo and colour -- domain and brand are independent concerns (XBR-35).

## Edge Cases

- **A client contact has a client-facing page already open when Nadia saves new branding** -- The already-loaded page is a snapshot from when it rendered; it does not live-update mid-view. The new logo/colour (or default) applies from the next page load or navigation onward.
- **An email is composed for sending at the same moment Nadia changes her branding** -- The email carries whichever branding was resolved at the moment it was actually composed and sent, not an earlier or later value; an email already delivered is never retroactively changed.
- **Nadia has set a logo but never set a colour (or vice versa)** -- Per the Independent per-field fallback rule, the set field shows her own value and the unset field shows its own neutral default; this is a valid, expected combination, not treated as incomplete.
- **Nadia resets to default while a client contact is mid-session in the portal** -- Consistent with the "page already open" edge case above: currently rendered pages are unaffected until reload; new page loads and any new email show the neutral default.
- **The freelancer account is deleted (FEAT-24) while a queued email referencing her Branding Profile has not yet sent** -- The email delivery process reads the neutral default when the underlying account and profile no longer exist, rather than failing outright; this is a defensive fallback and never surfaces an error to the recipient.
- **A custom domain (FEAT-27, Later) is added or removed** -- The resolved logo and colour are identical regardless of which address serves the page; only the address changes, never the branding this spec resolves.
- **The referral mark and the neutral default both appear together (no branding ever set)** -- Both render as designed; the neutral default is not treated as "no branding to show alongside," and the mark's presence is unaffected by whether branding is set.
- **Two client-facing surfaces from different consuming features render within the same client session (e.g., a portal page and a linked invoice)** -- Both resolve the same Branding Profile independently and show identical branding, since there is exactly one profile per Freelancer Account.

## Acceptance Criteria

**FEAT-19.SPEC-003-AC-01:** Given Nadia has a logo and colour saved, when Owen opens any client portal screen (FEAT-05), then the screen renders with Nadia's logo and colour applied.

**FEAT-19.SPEC-003-AC-02:** Given Nadia has never set any branding, when Priya opens a deliverable review screen (FEAT-06), then it renders with the clean neutral default logo treatment and palette, not a broken or generic-looking layout.

**FEAT-19.SPEC-003-AC-03:** Given Nadia has set a logo but no colour, when a client-facing invoice screen renders (FEAT-09), then it shows her logo alongside the neutral default palette.

**FEAT-19.SPEC-003-AC-04:** Given Nadia saves a new logo and colour, when the save completes, then the very next client-facing email FEAT-14 sends carries the new values, with no separate propagation delay.

**FEAT-19.SPEC-003-AC-05:** Given a client-facing proposal screen (FEAT-02) is already open in Owen's browser when Nadia saves new branding, when he continues viewing without reloading, then the page keeps showing the branding that was current when it loaded.

**FEAT-19.SPEC-003-AC-06:** Given Nadia resets her branding to default, when any client-facing screen or email renders or sends afterward, then it shows the clean neutral default, exactly as if branding had never been set.

**FEAT-19.SPEC-003-AC-07:** Given the referral mark (FEAT-33) appears on a client-facing page alongside Nadia's applied logo and colour, when the page renders, then the mark is shown without adopting her brand colour, without replacing her logo, and without her branding being hidden or resized.

**FEAT-19.SPEC-003-AC-08:** Given the referral mark appears on a page where branding was never set, when the page renders, then it appears alongside the neutral default exactly as it would alongside a freelancer's own branding.

**FEAT-19.SPEC-003-AC-09:** Given Dana opens a support session and views Branding Settings (FEAT-31), when the read-only screen loads via FEAT-19.SPEC-001, then it shows the same resolved logo and colour this spec defines, with no editing control.

**FEAT-19.SPEC-003-AC-10:** Given Owen or Priya attempts to change the applied branding directly, when the attempt is made, then no editing surface for it exists anywhere they can reach; the only editing surface in the product is FEAT-19.SPEC-001, which they cannot open.

**FEAT-19.SPEC-003-AC-11:** Given a freelancer's account is deleted while an already-queued client email has not yet sent, when the delivery process resolves branding for it, then it falls back to the neutral default rather than failing.

**FEAT-19.SPEC-003-AC-12:** Given a freelancer has both a shared default address and a verified custom domain (FEAT-27, Later), when a client-facing page is viewed at either address, then the resolved logo and colour are identical.

**FEAT-19.SPEC-003-AC-13:** Given Nadia's Branding Profile has a legibility-adjusted colour from FEAT-19.SPEC-002, when this spec resolves the applied colour, then it uses that already-adjusted value directly, performing no further validation or adjustment.

**FEAT-19.SPEC-003-AC-14:** Given two different client-facing features (for example, a portal page from FEAT-05 and an invoice from FEAT-09) render for the same freelancer's clients, when each resolves branding independently, then both show identical logo and colour, since exactly one Branding Profile exists per Freelancer Account.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
