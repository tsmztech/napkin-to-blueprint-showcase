---
document_type: spec
spec_type: screen
spec_id: FEAT-19.SPEC-001
spec_name: Branding Settings
spec_slug: branding-settings
parent_feature: FEAT-19
parent_feature_name: Freelancer Branding
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 24
---

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
