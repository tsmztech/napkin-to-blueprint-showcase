---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-19.SPEC-003
spec_name: Branding Application & Fallback Rule
spec_slug: branding-application-fallback-rule
parent_feature: FEAT-19
parent_feature_name: Freelancer Branding
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 11
acceptance_criteria_count: 14
---

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
