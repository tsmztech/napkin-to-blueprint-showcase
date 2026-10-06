# Feature Specification: Freelancer Branding

**Blueprint feature:** FEAT-19
**Priority tier:** Important
**Build order:** 013 of 33
**Depends on:** —
**Blueprint source:** `docs/blueprint/specifications/FEAT-19-freelancer-branding/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Branding Settings (Priority: P2)

Nadia uploads a logo, picks a brand colour, and can reset both to the neutral default; Dana views the same screen read-only inside a logged support session.

**Acceptance Scenarios:**

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

### User Story 2 - Branding Upload & Legibility Validation Rules (Priority: P2)

Governs the logo file size/format limits, the one-logo/one-colour-per-account limit, and the automatic legibility adjustment applied to a brand colour that would make text hard to read.

**Acceptance Scenarios:**

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

### User Story 3 - Branding Application & Fallback Rule (Priority: P2)

Governs how the freelancer's logo and colour -- or the clean neutral default when unset -- are applied to every client-facing screen and email, and how the referral mark coexists with them without overriding them.

**Acceptance Scenarios:**

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

### Edge Cases

- **FEAT-19.SPEC-001 (Branding Settings):** Saving different values from two sessions is last-write-wins with the earlier session stale until reload, and a failed logo upload keeps the previous logo and colour active everywhere with an inline error. Double taps on Save are ignored and unsaved changes trigger the discard confirmation. Source: `docs/blueprint/specifications/FEAT-19-freelancer-branding/FEAT-19.SPEC-001-branding-settings.md` (section: Edge Cases)
- **FEAT-19.SPEC-002 (Branding Upload & Legibility Validation Rules):** A logo file exactly at the size ceiling passes and one byte over fails, the format check inspects file content rather than the extension, and a brand colour exactly at the contrast ratio passes unadjusted (inclusive). A colour whose whole hue never clears the threshold is adjusted to the darkest legible shade so a legible value is always stored. Source: `docs/blueprint/specifications/FEAT-19-freelancer-branding/FEAT-19.SPEC-002-branding-upload-legibility-validation-rules.md` (section: Edge Cases)
- **FEAT-19.SPEC-003 (Branding Application & Fallback Rule):** Already-open client pages are snapshots and pick up new branding on the next load, and an email carries whichever branding was resolved when it was composed and sent. Logo and colour fall back independently (a logo set without a colour is valid), and a reset to default takes effect on new loads. Source: `docs/blueprint/specifications/FEAT-19-freelancer-branding/FEAT-19.SPEC-003-branding-application-fallback-rule.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-19.SPEC-001** (Branding Settings) as specified: Nadia uploads a logo, picks a brand colour, and can reset both to the neutral default; Dana views the same screen read-only inside a logged support session. Full spec: `docs/blueprint/specifications/FEAT-19-freelancer-branding/FEAT-19.SPEC-001-branding-settings.md`
- **FR-002**: The system MUST implement **FEAT-19.SPEC-002** (Branding Upload & Legibility Validation Rules) as specified: Governs the logo file size/format limits, the one-logo/one-colour-per-account limit, and the automatic legibility adjustment applied to a brand colour that would make text hard to read. Full spec: `docs/blueprint/specifications/FEAT-19-freelancer-branding/FEAT-19.SPEC-002-branding-upload-legibility-validation-rules.md`
- **FR-003**: The system MUST implement **FEAT-19.SPEC-003** (Branding Application & Fallback Rule) as specified: Governs how the freelancer's logo and colour -- or the clean neutral default when unset -- are applied to every client-facing screen and email, and how the referral mark coexists with them without overriding them. Full spec: `docs/blueprint/specifications/FEAT-19-freelancer-branding/FEAT-19.SPEC-003-branding-application-fallback-rule.md`

### Key Entities

- Branding Profile (create, update)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Logo uploads, brand colour settings and resets to default are each observable as distinct signals (branding_logo_uploaded, branding_color_set, branding_reset_to_default); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-27**: Brand colours are kept legible and screens never rely on colour alone. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-21**: Client-facing pages are expected to be interactive within roughly 2 seconds on a typical mobile connection. Full register: `docs/blueprint/features/assumptions-constraints.md`
