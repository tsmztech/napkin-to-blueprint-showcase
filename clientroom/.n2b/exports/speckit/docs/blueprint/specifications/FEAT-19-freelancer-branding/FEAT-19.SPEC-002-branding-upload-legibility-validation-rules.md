---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-19.SPEC-002
spec_name: Branding Upload & Legibility Validation Rules
spec_slug: branding-upload-legibility-validation-rules
parent_feature: FEAT-19
parent_feature_name: Freelancer Branding
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 14
acceptance_criteria_count: 14
---

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
