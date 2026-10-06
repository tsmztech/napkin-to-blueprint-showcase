# FEAT-02 — Availability & Working Hours Setup

This chapter covers Availability & Working Hours Setup (FEAT-02), a Core-tier feature. It carries 5 specifications carrying 82 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-02.SPEC-001 | Working Hours, Buffer, Notice & Horizon Setup | screen | 21 |
| FEAT-02.SPEC-002 | Per-Service Buffer Override | screen | 15 |
| FEAT-02.SPEC-003 | Availability Rule Versioning | automation | 11 |
| FEAT-02.SPEC-004 | Confirmed Booking Conflict Flagging | automation | 11 |
| FEAT-02.SPEC-005 | Availability Setup Validation & Limits | logic-rule | 24 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Availability & Working Hours Setup

## Summary

**Feature:** Availability & Working Hours Setup
**ID:** FEAT-02
**Description:** The Pro sets their recurring working hours and the buffer time they need between clients, forming the base schedule the availability engine works from.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Target Users & Roles states the Pro sets "working hours, buffer time between clients" as a core setup action. MVP: real-time availability has nothing to compute from without it.

**Key Capabilities:**
- Set weekly working hours (per day of week, with multiple windows per day allowed)
- Set default buffer time applied between consecutive bookings
- Override buffer time per service where a service genuinely needs more or less
- Set a minimum booking notice (how close to an appointment a client may still book) and a booking horizon (how far ahead clients may book)

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-02.SPEC-001 | Working Hours, Buffer, Notice & Horizon Setup | Screen | The Pro | Pro sets weekly working windows, default buffer, minimum booking notice, and booking horizon |
| FEAT-02.SPEC-002 | Per-Service Buffer Override | Screen | The Pro | Pro sets a buffer override for an individual service that needs more or less gap than the default |
| FEAT-02.SPEC-003 | Availability Rule Versioning | Automation | The Pro | System saves a new dated version of the Availability Rule on every save rather than overwriting the prior one |
| FEAT-02.SPEC-004 | Confirmed Booking Conflict Flagging | Automation | The Pro | System checks existing confirmed bookings against a newly saved Availability Rule and flags any that now fall outside working hours, without cancelling them |
| FEAT-02.SPEC-005 | Availability Setup Validation & Limits | Logic/Rule | The Pro | Validation rules governing working windows, buffer bounds, notice/horizon bounds, and timezone interpretation |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Set weekly working hours (per day, multiple windows) | FEAT-02.SPEC-001 | Primary purpose of the setup screen -- weekly grid with add/remove window per day | Phase 2 (Explicit) |
| Set default buffer time | FEAT-02.SPEC-001 | Single default-buffer field on the same setup screen | Phase 2 (Explicit) |
| Override buffer time per service | FEAT-02.SPEC-002 | Dedicated per-service override screen listing the Pro's services | Phase 2 (Explicit) |
| Set minimum booking notice and booking horizon | FEAT-02.SPEC-001 | Two additional fields on the same setup screen | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-02.SPEC-003 | Availability Rule Versioning | Phase 4 (Trigger-Response -- entity update) | The dependency map states the Availability Rule is "Updated by FEAT-02 (versioned by effective date)" -- an update that must never overwrite history, since past bookings need the rule version that was live when they were made. This is processing logic beyond a direct field write, so it is a standalone Automation. |
| FEAT-02.SPEC-004 | Confirmed Booking Conflict Flagging | Phase 4 (Trigger-Response -- entity update) / reinforced by Phase 6 (failure analysis of the "Alternate: Pro changes hours mid-week" flow) | The feature's own Primary Flows & Alternates state that already-confirmed bookings outside new hours "are never silently cancelled -- they remain honored and flagged for the Pro's attention," and XBR-11 makes this a cross-feature rule. This is a cross-entity side-effect (Availability Rule change affecting Booking records), so it is a standalone Automation rather than an inline screen behavior. |
| FEAT-02.SPEC-005 | Availability Setup Validation & Limits | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field names five-plus distinct rules (window start-before-end, no overlapping windows, buffer 0-120 minutes, notice 0-7 days, horizon 1 week-12 months, account-timezone interpretation) shared by both screens -- past the inline-validation threshold, so it becomes a standalone Logic/Rule spec. |

## Entity-Lifecycle Coverage Matrix

**Entity: Availability Rule**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-02.SPEC-001 | Pro's first save on the setup screen creates the initial Availability Rule; also created during the setup wizard (FEAT-15), which hands the Pro into this same screen | The "Empty" state (no hours set, cannot be booked) is owned by this Create path |
| Read (single) | FEAT-02.SPEC-001, FEAT-02.SPEC-002 | Both screens load the current (latest-effective) Availability Rule to display existing hours, default buffer, notice, horizon, and any per-service overrides | -- |
| Read (list) | N/A | The product definition gives the Pro no screen that browses past Availability Rule versions -- product-features.md's States field describes this feature as an instant, small-dataset setup screen with no history browser | Superseded versions are retained internally (for FEAT-03's and FEAT-30's conflict evaluation) but never surfaced as a browsable list; recorded as a non-goal below |
| Update | FEAT-02.SPEC-001, FEAT-02.SPEC-002 | Pro edits weekly windows, default buffer, per-service override, notice, or horizon and saves | Every update is versioned, not overwritten -- see next row |
| Delete/Archive | N/A | The dependency map's Availability Rule lifecycle states explicitly: "Deleted: N/A -- superseded by a newer version, never removed while past bookings reference it." No soft-or-hard delete path exists; a new version supersedes the old one (FEAT-02.SPEC-003), and the old version is retained indefinitely as long as any booking may reference it (SC-21 correctness bar) | Retention/purge is therefore an explicit non-goal, not an omission -- see Non-Goals |
| State Transition | N/A | The Availability Rule has no state machine of its own; version succession (via `effective_from`) is the only lifecycle mechanism, and it is fully covered by FEAT-02.SPEC-003 | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Service | FEAT-02.SPEC-002 | Lists the Pro's active services so the Pro can set a per-service buffer override against each one |
| Pro Account | FEAT-02.SPEC-001, FEAT-02.SPEC-005 | Reads the account timezone so every entered window is interpreted and displayed in the Pro's own timezone (XBR-25) |
| Booking | FEAT-02.SPEC-004 | Reads confirmed bookings to check each one against the newly saved Availability Rule and flag conflicts |

**Flagged discrepancy (not resolved here):** the Service entity's `buffer_override` field is described in the dependency map's Service lifecycle as "(set in FEAT-02)," yet the same Service lifecycle line lists only "Updated by FEAT-01" as the entity's updater. FEAT-02.SPEC-002 is the screen the Pro actually uses to set this field, so this Brief records the field as written by FEAT-02.SPEC-002 and flags the Service-entity lifecycle line for the Requirements Architect to reconcile -- per this feature's own decomposition rules, an interaction inconsistent with the dependency map is flagged, not silently resolved.

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro saves weekly hours, default buffer, notice, or horizon | Validate all entered values against the shared rule set | Standalone Logic/Rule | FEAT-02.SPEC-005 |
| Pro saves a per-service buffer override | Validate the override value against the same buffer bounds | Standalone Logic/Rule | FEAT-02.SPEC-005 |
| Pro's save passes validation | Create a new dated version of the Availability Rule (never overwrite) | Standalone Automation | FEAT-02.SPEC-003 |
| A new Availability Rule version is saved | Check confirmed bookings against the new rule; flag any that now fall outside working hours for the Pro's attention (without cancelling) | Standalone Automation | FEAT-02.SPEC-004 |
| A new Availability Rule version is saved | Bookable slots reflect the new rule immediately going forward | Cross-feature -- slot computation is owned by FEAT-03 | FEAT-03 responsibility |
| Pro saves successfully (no conflicts found) | Show success confirmation, remain on the setup screen | Inline in triggering screen | FEAT-02.SPEC-001 / SPEC-002 |
| Pro's save fails (e.g., connectivity) | Preserve entered values on-screen, offer retry | Inline in triggering screen | FEAT-02.SPEC-001 / SPEC-002 |
| Pro attempts to close a normally-working day for a one-off reason | Directed to Manual Time Blocking rather than editing the recurring rule | Cross-feature | FEAT-17 responsibility |

## Shared Context

**Shared Entities:**
- Availability Rule -- created and versioned by FEAT-02.SPEC-001/SPEC-003 (weekly windows, default buffer, minimum booking notice, booking horizon, effective_from); the per-service override value is set through FEAT-02.SPEC-002 but is flagged above as living on the Service entity rather than the Availability Rule itself.
- Service (referenced) -- FEAT-02.SPEC-002 reads the Pro's active service list and writes each service's `buffer_override` field; full Service lifecycle (name, price, duration, deposit rule) belongs to FEAT-01.
- Pro Account (referenced) -- FEAT-02.SPEC-001 and FEAT-02.SPEC-005 read the account's timezone; the Pro never sets timezone from within this feature (see Non-Goals).
- Booking (referenced) -- FEAT-02.SPEC-004 reads confirmed bookings' start times and durations to detect conflicts with a newly saved rule; it never writes to Booking records itself -- flagging and any resulting Pro action is surfaced and actioned through FEAT-30 (Pro Booking Management).

**Shared UI Patterns:**
- Weekly window editor -- used by FEAT-02.SPEC-001; a per-day list of start/end time pairs with add/remove controls, all values interpreted in the account timezone shown alongside the field.
- Buffer/notice/horizon numeric fields -- shared field pattern between FEAT-02.SPEC-001 (default buffer, notice, horizon) and FEAT-02.SPEC-002 (per-service override); same bounds validation (FEAT-02.SPEC-005), same "minutes/days/weeks" unit labeling, so Spec Writers should describe them identically across both screens.

**Shared Validation:**
- FEAT-02.SPEC-005 defines every validation and boundary rule (window ordering, no-overlap, buffer 0-120 minutes, notice 0-7 days, horizon 1 week-12 months, timezone interpretation). FEAT-02.SPEC-001 and FEAT-02.SPEC-002 both reference SPEC-005 for field validation rather than duplicating the rules.

## Internal Dependency Map

```
SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) -> [Pro taps Save] -> SPEC-005 (Availability Setup Validation & Limits) -> [valid] -> SPEC-003 (Availability Rule Versioning) -> [new version saved] -> SPEC-004 (Confirmed Booking Conflict Flagging)
SPEC-001 -> [validation fails] -> SPEC-001 [inline error, values preserved]
SPEC-002 (Per-Service Buffer Override) -> [Pro taps Save] -> SPEC-005 (Availability Setup Validation & Limits) -> [valid] -> SPEC-003 (Availability Rule Versioning) -> [new version saved] -> SPEC-004 (Confirmed Booking Conflict Flagging)
SPEC-001 -> [Pro navigates to per-service overrides] -> SPEC-002
SPEC-002 -> [Pro navigates back] -> SPEC-001
SPEC-004 -> [conflict found] -> [flag surfaced on Pro Booking Management dashboard, cross-feature to FEAT-30]
```

**Default Entry:** SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) -- the screen shown when the Pro navigates to availability setup, including the first-time entry from the setup wizard (FEAT-15).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-02.SPEC-001 | Inbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Wizard hands the Pro into this screen as the required hours step; if the Pro leaves mid-setup, the wizard resumes exactly here with services already saved | Pro continues setup after saving services |
| FEAT-02.SPEC-001 | Outbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Confirms working hours are set, satisfying one of the conditions FEAT-15 checks before the booking link can go live (XBR-26) | Pro completes and saves the hours step |
| FEAT-02.SPEC-001, FEAT-02.SPEC-002, FEAT-02.SPEC-003 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Every saved Availability Rule version is the primary input FEAT-03 combines with Time Blocks, Bookings, calendar busy time, and Recurring Series to compute open slots | Any hours/buffer/notice/horizon save |
| FEAT-02.SPEC-001 | Inbound | FEAT-17 (Manual Time Blocking) | A one-off closed day is handled there instead of by editing the recurring rule here | Pro wants to close a single day rather than change the weekly pattern |
| FEAT-02.SPEC-004 | Outbound | FEAT-30 (Pro Booking Management) | Flagged conflicting bookings are surfaced and resolved only through explicit Pro choice in Pro Booking Management, never auto-cancelled here (XBR-11) | A newly saved rule leaves a confirmed booking outside working hours |
| FEAT-02.SPEC-001, FEAT-02.SPEC-005 | Inbound | FEAT-27 (Pro Profile & Booking Page Settings) | Reads the Pro's account timezone, which FEAT-27 owns, to interpret and label every entered time (XBR-25) | Screen load and validation |
| FEAT-02.SPEC-001, FEAT-02.SPEC-002 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Every Pro-facing screen in this feature requires a signed-in Pro; anyone else is sent to sign-in (XBR-29) | Any navigation to this feature |
| FEAT-02.SPEC-002 | Outbound | FEAT-01 (Service & Pricing Management) | Writes the per-service buffer override onto the Service record that FEAT-01 otherwise owns (see flagged discrepancy in the Entity-Lifecycle section) | Pro sets or changes a per-service override |

## Non-Functional Notes

**Data volumes / growth:** Each Pro has exactly one active Availability Rule at a time plus a small, slowly-growing set of superseded versions retained for as long as any booking may reference them; this is a small dataset per account and stays well within the "few hundred pros, 100-500 clients, 20-40 bookings/week" scale set project-wide (assumptions-constraints.md ASMP-22).

**Responsiveness:** The setup screen is described as instant with a small dataset (product-features.md, States); saving hours or a buffer override should complete and be reflected in bookable slots immediately, with no perceptible wait, consistent with ASMP-27's rule that every waiting screen shows an in-place indicator rather than a blank page.

**Data sensitivity / privacy:** Low. The Pro's working pattern (Availability Rule) is private to the Pro; clients never see the rule itself, only the resulting open times it produces (product-features.md, Data Notes). No third-party personal data is involved.

**Compliance flags:** N/A -- no health, financial, or identity data is captured by this feature; the product-wide correctness bar (ASMP-26: never silently double-book) is the operative non-functional constraint, and it is met structurally here by never letting a rule change silently cancel a confirmed booking (FEAT-02.SPEC-004).

## Non-Goals

- **Version-history browsing screen for past Availability Rules** -- Excluded per product-features.md's States field, which describes this feature as a single instant setup screen with no version browser; superseded versions are kept only for internal conflict evaluation (FEAT-02.SPEC-004, FEAT-03), never surfaced to the Pro as a browsable list.
- **Automatic purge of superseded Availability Rule versions** -- Intentional lifecycle decision surfaced by the CRUD matrix: the dependency map states Availability Rules are "never removed while past bookings reference it," and SC-21 sets correctness over convenience as the product's bar from MVP onward, so retention has no purge window pending a booking's full lifetime.
- **Setting or changing the Pro's account timezone from this feature** -- Excluded per XBR-25, which names FEAT-27 as the sole owner of timezone (and currency) settings; this feature only reads and interprets hours in whatever timezone FEAT-27 has set.
- **Per-staff or per-chair working-hours variants** -- Excluded per scope-boundaries.md SC-01: the product is strictly single-operator, so this feature manages exactly one working-hours rule set for the one Pro on the account, never a roster of staff schedules.
- **Client-facing display of the Availability Rule itself** -- Excluded per the Access Matrix (user-persona.md): Clients have no access to Service & Availability Setup and see only its effect (open slots) through FEAT-03; this feature has no client-facing surface at all.



# Screen Spec: Working Hours, Buffer, Notice & Horizon Setup

## Overview

**Name:** Working Hours, Buffer, Notice & Horizon Setup
**ID:** FEAT-02.SPEC-001
**Type:** Screen
**Purpose:** Talia (the Pro) sets her recurring weekly working windows, default buffer time between bookings, minimum booking notice, and booking horizon — the base schedule the availability engine computes from.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup

## Scope and Non-Goals

**In Scope:**
- Setting weekly working hours per day of week, with multiple non-overlapping windows allowed per day
- Setting a single default buffer time applied between consecutive bookings
- Setting minimum booking notice and booking horizon
- Creating the initial Availability Rule on first save, and saving every subsequent edit as a new dated version (via FEAT-02.SPEC-003)
- Navigating to the per-service buffer override screen (FEAT-02.SPEC-002)
- Navigating to the time block create screen (FEAT-17.SPEC-001) to close a single day

**Non-Goals:**
- Per-service buffer overrides -- handled entirely on FEAT-02.SPEC-002, reached from this screen
- Field-level and cross-field validation logic -- owned by FEAT-02.SPEC-005 (Availability Setup Validation & Limits); this screen only displays the outcome
- Browsing or restoring a past Availability Rule version -- excluded per feature-overview.md's Non-Goals: the product definition gives the Pro no version-history browser; superseded versions are retained only for internal conflict evaluation
- Setting the Pro's account timezone -- excluded per XBR-25 and feature-overview.md's Non-Goals: FEAT-27 is the sole owner of timezone; this screen only reads and displays it
- One-off closure of a normally-working day -- not edited here; the "Close a single day" row hands the Pro to FEAT-17.SPEC-001 (Manual Time Blocking), per feature-overview.md's Primary Flows & Alternates

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard) -- setup step: hours | Pro continues setup after saving services | None -- screen opens in its Empty state (no Availability Rule yet exists for this account) |
| Default entry (Pro navigation, e.g. from settings or a dashboard prompt) | Pro navigates directly to availability setup | Loads the current (latest-effective) Availability Rule, if one exists |
| FEAT-02.SPEC-002 (Per-Service Buffer Override) | Pro taps back/navigate | Returns to this screen showing the current Availability Rule values as last saved |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All actions -- edit weekly windows, default buffer, minimum booking notice, booking horizon; navigate to per-service overrides; save | -- |
| The Client (Riley) | No | No | Clients have no access to Service & Availability Setup (Access Matrix, user-persona.md); there is no client-facing navigation path into this screen at all |
| Platform Operator (Support) | Full screen, read-only | No -- all edit and save controls are hidden | A direct save attempt is not reachable from the UI (controls are hidden, not merely disabled); if attempted through a stale or replayed request, the response is "Support access is read-only and cannot make changes to this account." (XBR-24) |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed sign-in never reveals whether an account exists (XBR-29) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any unsaved edits on the form are preserved in the browser and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Working Hours" with a back arrow (returns to the Pro's settings or, when reached from FEAT-15, advances the wizard to its next step) and a "Save" action button (right-aligned).

**Body, in order:**
- **Weekly window editor:** One row per day of week (Sunday through Saturday), each showing its list of working windows as start-time/end-time field pairs, an "Add window" control per day, and a remove control on each window past the first. All times are labeled with the Pro's account timezone (read from Pro Account, FEAT-27) shown once at the top of this section (e.g., "All times shown in America/New_York").
- **Default buffer field:** A single numeric field labeled "Buffer time between bookings (minutes)," applied to every booking that has no per-service override.
- **Minimum booking notice field:** A numeric/unit field labeled "How close to an appointment can a client still book?" (value plus a days/hours unit).
- **Booking horizon field:** A numeric/unit field labeled "How far ahead can clients book?" (value plus a weeks/months unit).
- **Per-service overrides entry point:** A row labeled "Buffer overrides by service" with a "Manage" control that navigates to FEAT-02.SPEC-002.
- **Close a single day entry point:** A row labeled "Need to close just one day? Block time" with a "Block time" control that navigates to FEAT-17.SPEC-001 (Create/Edit Time Block) in create mode, pre-focused on the date field.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** The weekly window editor stacks one day per row, full width, with window pairs wrapping to a second line when both fields cannot fit one row. Save remains in the header.
- **Medium size class and above:** The weekly window editor and the buffer/notice/horizon fields render in a single centered column capped at a consistent platform-wide form width (exact value is the design layer's decision); no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the Pro's settings (or the wizard's next step if entered from FEAT-15) | Screen closes | Standard transition |
| "Add window" (per day) | Tap | Adds a new empty start/end field pair under that day | New window row appears | New fields shown empty, focus moves to the new start-time field |
| Remove window control | Tap | Removes that window from the day | Window row disappears | Remaining windows re-flow upward |
| Start time field | Select/type | Captures the window's start time | Field shows entered value | Standard input state; validated via FEAT-02.SPEC-005 |
| End time field | Select/type | Captures the window's end time | Field shows entered value | Standard input state; validated via FEAT-02.SPEC-005 |
| Default buffer field | Type | Captures the default buffer value in minutes | Field shows entered value | Validated via FEAT-02.SPEC-005 |
| Minimum booking notice field | Type/select | Captures the notice value and unit | Field shows entered value | Validated via FEAT-02.SPEC-005 |
| Booking horizon field | Type/select | Captures the horizon value and unit | Field shows entered value | Validated via FEAT-02.SPEC-005 |
| "Manage" (per-service overrides) | Tap | Navigate to FEAT-02.SPEC-002 (Per-Service Buffer Override) | Screen closes | Standard transition; current unsaved edits on this screen prompt the discard dialog first if any exist |
| "Block time" (close a single day) | Tap | Navigate to FEAT-17.SPEC-001 (Create/Edit Time Block) in create mode, pre-focused on the date field | Screen closes | Standard transition; current unsaved edits on this screen prompt the discard dialog first if any exist |
| Save button | Tap | 1. Validate all fields via FEAT-02.SPEC-005. 2. If valid, trigger FEAT-02.SPEC-003 (Availability Rule Versioning), which in turn triggers FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging). 3. On success, show confirmation and remain on screen. | Button shows loading state during save | Success: "Hours saved" confirmation banner, remains on screen with saved values shown. Failure: inline field errors (validation) or a retry banner (save failure). |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> each day's windows in calendar order (start, end, remove, add) -> default buffer -> minimum booking notice -> booking horizon -> "Manage" per-service overrides -> "Block time" -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Hours saved" confirmation is announced on success; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Adding and removing windows, and every other action on this screen, are reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (no hours set) | Every day shows zero windows and a guided prompt ("Add your first working window to start taking bookings"); default buffer, notice, and horizon show their defaults; Save is enabled | A brand-new Pro Account with no Availability Rule yet | Pro adds at least one window and saves |
| Filling | Form fields contain entered values; Save enabled | Pro edits any field | Pro taps Save or navigates away |
| Validating | Save button shows a loading spinner | Pro taps Save | Validation (FEAT-02.SPEC-005) completes (pass or fail) |
| Validation Error | Failed field(s) highlighted with their error messages shown inline | Validation fails | Pro corrects the field(s) and re-triggers validation |
| Saving | Save button shows a loading spinner, form fields disabled | Validation passes | FEAT-02.SPEC-003 completes or fails |
| Success | Confirmation banner "Hours saved"; form shows the saved values; remains on this screen | Save completes successfully | Banner dismisses after a few seconds or on next edit |
| Error | Error banner "Could not save your hours. Check your connection and try again." with a Retry action; all entered values remain on screen | The save operation (FEAT-02.SPEC-003) fails | Pro taps Retry or navigates away |
| Offline/Degraded | N/A -- this is a setup screen used between clients on a stable connection, not an in-the-moment mobile flow (product-features.md, FEAT-02 States); a connection loss during save surfaces through the ordinary Error state rather than a distinct offline mode | -- | -- |

## Validation Rules

Validation governed by FEAT-02.SPEC-005 (Availability Setup Validation & Limits). See that spec for all field-level and cross-field rules (window ordering, no-overlap, buffer bounds, notice bounds, horizon bounds, timezone interpretation). This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (direct entry) | Pro's settings screen | FEAT-27 (Pro Profile & Booking Page Settings) |
| Back arrow tap (entered from wizard) | Next wizard step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| "Manage" per-service overrides tap | FEAT-02.SPEC-002 (Per-Service Buffer Override) | -- |
| "Block time" (close a single day) tap | FEAT-17.SPEC-001 (Create/Edit Time Block) | FEAT-17 (Manual Time Blocking) |
| Successful save | Remains on this screen with the Success state shown | -- |

## Data Model

**Creates:** Availability Rule -- on the Pro's first save, sets weekly_windows, default_buffer, minimum_booking_notice, booking_horizon; effective_from is set by FEAT-02.SPEC-003.
**Reads:** Availability Rule -- the current (latest-effective) version's weekly_windows, default_buffer, minimum_booking_notice, and booking_horizon, to pre-fill the form. Pro Account -- timezone (read-only, for interpreting and labeling every entered time, per XBR-25).
**Updates:** Availability Rule -- weekly_windows, default_buffer, minimum_booking_notice, booking_horizon on every subsequent save; every update is versioned (never overwritten in place) by FEAT-02.SPEC-003.
**Deletes:** None -- superseded versions are retained, never deleted, per feature-overview.md's Non-Goals and the dependency map's Availability Rule lifecycle.

## Business Rules

- Every save is validated against FEAT-02.SPEC-005 before FEAT-02.SPEC-003 is triggered -- the Pro cannot save invalid values.
- A passing save always creates a new dated Availability Rule version rather than overwriting the prior one (FEAT-02.SPEC-003).
- Every new version triggers a check of existing confirmed bookings against the new rule (FEAT-02.SPEC-004); a conflicting booking is never silently cancelled -- it is flagged for the Pro's attention on Pro Booking Management (XBR-11).
- All entered and displayed times are interpreted and labeled in the Pro's account timezone, which this screen reads but never sets (XBR-25).
- Minimum booking notice and booking horizon limit every client-facing booking path computed downstream by FEAT-03 (XBR-03); the Pro alone may book inside notice or beyond horizon through Pro Booking Management.
- Completing this step (along with the other required setup steps) satisfies one of the conditions FEAT-15 checks before the Pro's booking link can go live (XBR-26).
- A one-off closed day is handled through Manual Time Blocking (FEAT-17) rather than by editing the recurring weekly windows here; the "Block time" row on this screen navigates to FEAT-17.SPEC-001 for that purpose.
- Every navigation to this screen requires a signed-in Pro (XBR-29).

## Edge Cases

- **Pro navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save your hours. Check your connection and try again." with a Retry button; all entered values remain on screen.
- **Pro enters overlapping windows on the same day** -- Caught by FEAT-02.SPEC-005 validation; Save does not proceed until resolved.
- **Save succeeds but the resulting conflict check (FEAT-02.SPEC-004) finds a conflicting confirmed booking** -- This screen still shows its own "Hours saved" success state; the conflict itself surfaces separately, on Pro Booking Management's attention list, not as an error on this screen.
- **Two Pro sessions (e.g., phone and desktop) save different edits to the same Availability Rule at nearly the same time** -- Last-write-wins between the Pro's own sessions, per the dependency map's Contention note for Availability Rule; each save independently creates its own new version, and the later save's version becomes the latest-effective one.
- **Pro's account timezone changes (via FEAT-27) while this screen is open with unsaved edits** -- The screen re-reads the current timezone at save time and labels all times accordingly; if the timezone changed since the values were entered, the Pro is shown a one-time notice that the displayed timezone label has updated before the save proceeds.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-002 (Per-Service Buffer Override) | Navigation (outbound), Navigation (inbound) | Pro navigates to per-service overrides from here and back |
| FEAT-02.SPEC-005 (Availability Setup Validation & Limits) | References (inbound) | Validation rules applied to every field on save |
| FEAT-02.SPEC-003 (Availability Rule Versioning) | Triggers (outbound) | A passing save triggers creation of a new dated Availability Rule version |
| FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging) | Triggers (outbound, indirect via FEAT-02.SPEC-003) | Every new version saved through this screen triggers the conflict check |
| FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound), Navigation (outbound) | Wizard hands the Pro in as its hours step and resumes here if setup is left mid-way; completing this step advances the wizard |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Reads the Pro's account timezone to interpret and label every entered time |
| FEAT-17.SPEC-001 (Create/Edit Time Block) -- within FEAT-17 (Manual Time Blocking) | Navigation (outbound) | The "Block time" row hands the Pro to the time block create form (create mode, date field focused) to close a single day instead of editing the weekly windows here |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) -- within FEAT-17 (Manual Time Blocking) | References (informational) | Recurring closures are created through FEAT-17, not edited here |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | References (inbound) | Requires a signed-in Pro for any access |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| availability_hours_updated | count of working windows, count of days with at least one window | A save completes successfully | supports success-metrics.md: "Availability Setup Accuracy" |
| buffer_time_updated | new default buffer value (minutes), previous value | A save completes successfully with a changed default_buffer | supports success-metrics.md: "Availability Setup Accuracy" |
| notice_horizon_updated | new minimum_booking_notice, new booking_horizon | A save completes successfully with a changed notice or horizon value | supports success-metrics.md: "Availability Setup Accuracy" |
| availability_setup_validation_failed | field(s) in error | Save is blocked by a validation failure | supports success-metrics.md: "Availability Setup Accuracy" (a validation failure reaching this screen after prior saves indicates the Pro's intent and the displayed offer of hours are diverging) |

## Acceptance Criteria

**FEAT-02.SPEC-001-AC-01:** Given Talia is on the Working Hours screen with no Availability Rule yet, then every day shows zero windows and a guided prompt to add her first working window.

**FEAT-02.SPEC-001-AC-02:** Given Talia is on the Working Hours screen, when she taps "Add window" under Tuesday, then a new empty start/end field pair appears under Tuesday with focus on the new start-time field.

**FEAT-02.SPEC-001-AC-03:** Given Talia has two windows under Monday, when she taps the remove control on the second window, then that window disappears and the first window remains.

**FEAT-02.SPEC-001-AC-04:** Given Talia enters a start time after its end time for a window, when the field loses focus, then FEAT-02.SPEC-005 flags the error inline on that window.

**FEAT-02.SPEC-001-AC-05:** Given Talia sets her default buffer to 15 minutes, when she taps Save and validation passes, then FEAT-02.SPEC-003 creates a new Availability Rule version with default_buffer set to 15.

**FEAT-02.SPEC-001-AC-06:** Given Talia sets her minimum booking notice to 2 days and her booking horizon to 10 weeks, when she taps Save and validation passes, then the new Availability Rule version reflects both values.

**FEAT-02.SPEC-001-AC-07:** Given Talia is on the Working Hours screen, when she taps "Manage" under per-service overrides, then she is navigated to FEAT-02.SPEC-002.

**FEAT-02.SPEC-001-AC-08:** Given Talia taps Save with a valid form, then the Save button shows a loading state, FEAT-02.SPEC-003 runs, and on success a "Hours saved" banner appears while she remains on this screen.

**FEAT-02.SPEC-001-AC-09:** Given Talia taps Save while a prior save for the same edit is still in progress, when she taps Save a second time, then the second tap is ignored and the button remains in its loading state.

**FEAT-02.SPEC-001-AC-10:** Given Talia's save fails due to a connectivity error, then an error banner reading "Could not save your hours. Check your connection and try again." appears with a Retry action, and all her entered values remain on screen.

**FEAT-02.SPEC-001-AC-11:** Given Talia has unsaved changes on this screen, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-02.SPEC-001-AC-12:** Given Talia saves a new Availability Rule version that leaves an existing confirmed booking outside the new hours, then this screen still shows its own "Hours saved" success state, and the conflict is surfaced separately on Pro Booking Management via FEAT-02.SPEC-004, never as an error here.

**FEAT-02.SPEC-001-AC-13:** Given Talia saves conflicting edits from two signed-in sessions at nearly the same time, then the later save's version becomes the latest-effective Availability Rule version, consistent with last-write-wins.

**FEAT-02.SPEC-001-AC-14:** Given Talia enters her hours, then every displayed and entered time is shown labeled with her account timezone as set in FEAT-27.

**FEAT-02.SPEC-001-AC-15:** Given Riley (the Client) has no navigational path to this screen, then no client-facing entry point into Working Hours exists anywhere in the product.

**FEAT-02.SPEC-001-AC-16:** Given Platform Operator (Support) opens Talia's account for troubleshooting, when Support views this screen, then all fields are shown read-only and no Save control is visible.

**FEAT-02.SPEC-001-AC-17:** Given a visitor who is not signed in as a Pro attempts to reach this screen, then they are redirected to the Pro sign-in screen without any indication of whether an account exists.

**FEAT-02.SPEC-001-AC-18:** Given Talia's session expires while she has unsaved edits on this screen, when the expiry is detected, then a dialog reading "Your session has expired. Sign in to continue." appears, and her unsaved edits are restored after she signs back in.

**FEAT-02.SPEC-001-AC-19:** Given Talia arrives at this screen from FEAT-15's setup wizard, when she completes and saves her hours, then the wizard advances to its next step.

**FEAT-02.SPEC-001-AC-20:** Given Talia has an existing Availability Rule, when she opens this screen through direct navigation (not the wizard), then the form pre-fills with the current (latest-effective) version's weekly windows, default buffer, minimum booking notice, and booking horizon.

**FEAT-02.SPEC-001-AC-21:** Given Talia is on the Working Hours screen, when she taps "Block time" under "Need to close just one day?", then she is navigated to FEAT-17.SPEC-001 in create mode with the date field focused (after the discard dialog if she has unsaved edits).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 11 | 11 |
| States | 8 (empty, filling, validating, validation error, saving, success, error, offline/degraded) | 8 |
| Business Rules | 8 | 8 |
| Edge Cases | 7 | 7 |



# Screen Spec: Per-Service Buffer Override

## Overview

**Name:** Per-Service Buffer Override
**ID:** FEAT-02.SPEC-002
**Type:** Screen
**Purpose:** Talia (the Pro) sets a buffer-time override for an individual service that genuinely needs more or less gap than her default buffer.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup

## Scope and Non-Goals

**In Scope:**
- Listing the Pro's active services with their current buffer state (using the default, or overridden)
- Setting, changing, or clearing a buffer override for an individual service
- Validating the override value against the same bounds as the default buffer

**Non-Goals:**
- Editing a service's name, price, duration, or deposit rule -- owned entirely by FEAT-01 (Service & Pricing Management); this screen touches only the buffer_override field
- Setting the account-wide default buffer, minimum booking notice, or booking horizon -- handled on FEAT-02.SPEC-001, which this screen is reached from
- Archiving or reordering services -- owned by FEAT-01; this screen only lists a Pro's currently active services for the purpose of setting their buffer override
- Field-level and cross-field validation logic -- owned by FEAT-02.SPEC-005 (Availability Setup Validation & Limits); this screen only displays the outcome

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Pro taps "Manage" under per-service overrides | None -- screen loads the Pro's current active service list and each service's current buffer_override, if any |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All actions -- set, change, or clear a per-service buffer override; save | -- |
| The Client (Riley) | No | No | Clients have no access to Service & Availability Setup (Access Matrix, user-persona.md); there is no client-facing navigation path into this screen at all |
| Platform Operator (Support) | Full screen, read-only | No -- all edit and save controls are hidden | A direct save attempt is not reachable from the UI (controls are hidden, not merely disabled); if attempted through a stale or replayed request, the response is "Support access is read-only and cannot make changes to this account." (XBR-24) |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed sign-in never reveals whether an account exists (XBR-29) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any unsaved edits on the form are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Buffer Overrides by Service" with a back arrow (returns to FEAT-02.SPEC-001) and a "Save" action button (right-aligned).

**Body:** A list, one row per active service in the Pro's current display order (FEAT-01), each row showing:
- Service name
- A numeric buffer-override field, pre-filled with the service's current override value if one exists, otherwise shown empty with placeholder text "Using default ({default_buffer} min)"
- A "Clear override" control, shown only on rows that currently have an override set, which resets the row to use the account default

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Service rows stack full width, one per row, with the service name above its buffer field.
- **Medium size class and above:** Service rows render as a single table with the service name and buffer field side by side, capped at a consistent platform-wide form width; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-02.SPEC-001 | Screen closes | Standard transition; unsaved edits prompt the discard dialog first if any exist |
| Buffer-override field (per service) | Type | Captures the override value in minutes for that service | Field shows entered value | Validated via FEAT-02.SPEC-005 |
| "Clear override" control (per service) | Tap | Clears the override so the service falls back to the account default | Field returns to its empty, placeholder state | Row shows "Using default ({default_buffer} min)" |
| Save button | Tap | 1. Validate every entered override via FEAT-02.SPEC-005. 2. If valid, trigger FEAT-02.SPEC-003 (Availability Rule Versioning), which in turn triggers FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging). 3. On success, show confirmation and remain on screen. | Button shows loading state during save | Success: "Overrides saved" confirmation banner. Failure: inline field errors (validation) or a retry banner (save failure). |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> each service row in display order (buffer field, then clear-override control when present) -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Overrides saved" confirmation is announced on success; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen, including clearing an override, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (no active services) | A guided message: "Add a service first to set buffer overrides" with a link to FEAT-01 | The Pro has zero active services | The Pro adds at least one active service and returns here |
| Filling | Service list shown with any entered edits; Save enabled | Pro edits any override field | Pro taps Save or navigates away |
| Validating | Save button shows a loading spinner | Pro taps Save | Validation (FEAT-02.SPEC-005) completes (pass or fail) |
| Validation Error | Failed field(s) highlighted with their error messages shown inline | Validation fails | Pro corrects the field(s) and re-triggers validation |
| Saving | Save button shows a loading spinner, form fields disabled | Validation passes | FEAT-02.SPEC-003 completes or fails |
| Success | Confirmation banner "Overrides saved"; remains on this screen | Save completes successfully | Banner dismisses after a few seconds or on next edit |
| Error | Error banner "Could not save your overrides. Check your connection and try again." with a Retry action; all entered values remain on screen | The save operation (FEAT-02.SPEC-003) fails | Pro taps Retry or navigates away |
| Offline/Degraded | N/A -- this is a setup screen used between clients on a stable connection, not an in-the-moment mobile flow, consistent with FEAT-02.SPEC-001's own stance | -- | -- |

## Validation Rules

Validation governed by FEAT-02.SPEC-005 (Availability Setup Validation & Limits). See that spec for the buffer-override bounds (the same bounds as the account default buffer). This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | -- |
| "Add a service" link (Empty state) | Service list / add-service screen | FEAT-01 (Service & Pricing Management) |
| Successful save | Remains on this screen with the Success state shown | -- |

## Data Model

**Creates:** None -- this screen never creates a Service record.
**Reads:** Service -- name, display_order, status (Active only) for the list; existing buffer_override values to pre-fill each row. Availability Rule -- default_buffer, to show as the placeholder fallback value on rows with no override.
**Updates:** Service -- buffer_override field only, per service, on save. (Flagged discrepancy, per feature-overview.md: the dependency map's Service lifecycle line lists only FEAT-01 as an updater of Service; this Brief records buffer_override as written by this screen and flags the line for the Requirements Architect to reconcile.)
**Deletes:** None.

## Business Rules

- Every entered override is validated against FEAT-02.SPEC-005's buffer bounds before FEAT-02.SPEC-003 is triggered -- the Pro cannot save an out-of-bounds override.
- A passing save creates a new dated Availability Rule version through FEAT-02.SPEC-003, exactly as a save on FEAT-02.SPEC-001 does, even though this screen's own writes land on the Service entity.
- Every new version triggers a check of existing confirmed bookings against the new rule (FEAT-02.SPEC-004); a conflicting booking is never silently cancelled -- it is flagged for the Pro's attention on Pro Booking Management (XBR-11).
- A service with no override uses the account's default_buffer, read from the current Availability Rule.
- Clearing an override removes it entirely rather than setting it to zero -- the row reverts to following the account default, including any future default changes.
- Only the Pro's currently active services are listed; an archived service's prior override is retained on its own record but not shown or editable here, consistent with FEAT-01's archive behavior.
- Every navigation to this screen requires a signed-in Pro (XBR-29).

## Edge Cases

- **Pro navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save your overrides. Check your connection and try again." with a Retry button; all entered values remain on screen.
- **Pro enters an override equal to the account default** -- Accepted; the override is still stored explicitly and takes precedence even if the account default later changes.
- **A service is archived by the Pro (via FEAT-01) while this screen is open with unsaved edits for it** -- That service's row is removed on the next load; any unsaved edit for it is discarded silently since the service is no longer bookable.
- **Two Pro sessions save different overrides for the same service at nearly the same time** -- Last-write-wins between the Pro's own sessions, consistent with the dependency map's Contention note for Service: the two screens (FEAT-01 and this one) write disjoint fields, so a concurrent FEAT-01 edit to name/price/duration never conflicts with this screen's buffer_override write.
- **Pro's account has services created after this screen was last opened** -- The list reflects the Pro's current active service set on every fresh load; a newly added service appears with no override (using the default) the next time this screen opens.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Navigation (inbound), Navigation (outbound) | Pro arrives from and returns to that screen |
| FEAT-02.SPEC-005 (Availability Setup Validation & Limits) | References (inbound) | Buffer-override bounds applied on save |
| FEAT-02.SPEC-003 (Availability Rule Versioning) | Triggers (outbound) | A passing save triggers creation of a new dated Availability Rule version |
| FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging) | Triggers (outbound, indirect via FEAT-02.SPEC-003) | Every new version saved through this screen triggers the conflict check |
| FEAT-01 (Service & Pricing Management) | References (inbound), Navigation (outbound) | Reads the Pro's active service list and display order; the Empty state links there to add a first service |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | References (inbound) | Requires a signed-in Pro for any access |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| service_buffer_override_set | service reference, override value (minutes) | A save completes successfully with a new or changed override | supports success-metrics.md: "Availability Setup Accuracy" |
| service_buffer_override_cleared | service reference, prior override value | A save completes successfully clearing an existing override | supports success-metrics.md: "Availability Setup Accuracy" |
| service_buffer_override_validation_failed | service reference, entered value | Save is blocked by a validation failure on an override field | supports success-metrics.md: "Availability Setup Accuracy" |

## Acceptance Criteria

**FEAT-02.SPEC-002-AC-01:** Given Talia has three active services and none has an override set, then this screen lists all three with each buffer field showing the "Using default ({default_buffer} min)" placeholder.

**FEAT-02.SPEC-002-AC-02:** Given Talia enters 30 minutes as the override for her "Full Set" service, when she taps Save and validation passes, then FEAT-02.SPEC-003 creates a new Availability Rule version reflecting that service's buffer_override as 30.

**FEAT-02.SPEC-002-AC-03:** Given Talia has an existing override on a service, when she taps "Clear override" and saves, then that service's row shows the default-buffer placeholder again and its buffer_override field is cleared.

**FEAT-02.SPEC-002-AC-04:** Given Talia enters an override value outside the bounds defined by FEAT-02.SPEC-005, when the field loses focus, then an inline error appears and Save does not proceed until it is corrected.

**FEAT-02.SPEC-002-AC-05:** Given Talia has zero active services, then this screen shows the guided message "Add a service first to set buffer overrides" with a link into FEAT-01.

**FEAT-02.SPEC-002-AC-06:** Given Talia taps Save with valid overrides entered, then the Save button shows a loading state, FEAT-02.SPEC-003 runs, and on success an "Overrides saved" banner appears while she remains on this screen.

**FEAT-02.SPEC-002-AC-07:** Given Talia taps Save while a prior save is still in progress, when she taps Save a second time, then the second tap is ignored and the button remains in its loading state.

**FEAT-02.SPEC-002-AC-08:** Given Talia's save fails due to a connectivity error, then an error banner reading "Could not save your overrides. Check your connection and try again." appears with a Retry action, and all entered values remain on screen.

**FEAT-02.SPEC-002-AC-09:** Given Talia has unsaved changes on this screen, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-02.SPEC-002-AC-10:** Given Talia archives a service (via FEAT-01) while this screen has an unsaved override for it, when the screen next loads, then that service's row no longer appears and the discarded edit has no effect.

**FEAT-02.SPEC-002-AC-11:** Given Riley (the Client) has no navigational path to this screen, then no client-facing entry point into Per-Service Buffer Override exists anywhere in the product.

**FEAT-02.SPEC-002-AC-12:** Given Platform Operator (Support) opens Talia's account for troubleshooting, when Support views this screen, then all fields are shown read-only and no Save control is visible.

**FEAT-02.SPEC-002-AC-13:** Given a visitor who is not signed in as a Pro attempts to reach this screen, then they are redirected to the Pro sign-in screen without any indication of whether an account exists.

**FEAT-02.SPEC-002-AC-14:** Given Talia's session expires while she has unsaved edits on this screen, when the expiry is detected, then a dialog reading "Your session has expired. Sign in to continue." appears, and her unsaved edits are restored after she signs back in.

**FEAT-02.SPEC-002-AC-15:** Given Talia saves an override that, combined with a booking's fixed duration, leaves an existing confirmed booking outside the new rule's fit, then this screen still shows its own "Overrides saved" success state, and the conflict is surfaced separately on Pro Booking Management via FEAT-02.SPEC-004, never as an error here.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 8 (empty, filling, validating, validation error, saving, success, error, offline/degraded) | 8 |
| Business Rules | 7 | 7 |
| Edge Cases | 6 | 6 |



# Automation Spec: Availability Rule Versioning

## Overview

**Name:** Availability Rule Versioning
**ID:** FEAT-02.SPEC-003
**Type:** Automation
**Purpose:** System saves every passing Availability Rule edit as a new dated version rather than overwriting the prior one, so past bookings keep the rule that was live when they were made.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup

## Scope and Non-Goals

**In Scope:**
- Creating the initial Availability Rule on a Pro's first passing save
- Creating a new, dated version of the Availability Rule on every subsequent passing save, from either FEAT-02.SPEC-001 or FEAT-02.SPEC-002
- Setting the new version's effective_from date and retaining every prior version
- Triggering the confirmed-booking conflict check (FEAT-02.SPEC-004) once the new version is committed

**Non-Goals:**
- Validating the entered values -- owned entirely by FEAT-02.SPEC-005; this automation only runs after validation has already passed
- Checking existing confirmed bookings against the new version -- owned by FEAT-02.SPEC-004, which this automation triggers but does not itself perform
- Surfacing a browsable history of past versions to the Pro -- excluded per feature-overview.md's Non-Goals: the product definition gives the Pro no version-history browser; superseded versions are retained only for internal conflict evaluation (this spec, FEAT-03)
- Purging superseded versions -- excluded per feature-overview.md's Non-Goals and scope-boundaries.md SC-22: versions are retained for as long as any booking may reference them, with no purge window

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro's save passes validation (weekly hours, default buffer, notice, horizon) | FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Fires only after FEAT-02.SPEC-005 validation succeeds | Full set of entered field values: weekly_windows, default_buffer, minimum_booking_notice, booking_horizon |
| Pro's save passes validation (per-service buffer override) | FEAT-02.SPEC-002 (Per-Service Buffer Override) | Fires only after FEAT-02.SPEC-005 validation succeeds | The current Availability Rule's other fields (unchanged) plus the entered Service.buffer_override value(s) |

## Processing Logic

1. Receive the validated field values from the triggering screen (either the full Availability Rule field set from FEAT-02.SPEC-001, or the per-service override value(s) from FEAT-02.SPEC-002).
2. Read the current (latest-effective) Availability Rule version for this Pro Account, if one exists.
3. Construct the new version by combining the triggering screen's changed fields with every unchanged field carried forward from the current version (e.g., a FEAT-02.SPEC-002 save carries forward weekly_windows, default_buffer, minimum_booking_notice, and booking_horizon unchanged, adding only the new Service.buffer_override).
4. Set the new version's effective_from to the moment the save commits.
5. Write the new version as a wholly new, additional Availability Rule record -- the prior version is never modified or removed; it remains retrievable by its own effective_from.
6. Mark the newly written version as the Pro Account's current (latest-effective) Availability Rule.
7. Trigger FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging) against the newly written version.
8. Return the outcome (success or failure) to the triggering screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| First version created | No Availability Rule existed for this Pro Account before this save | A new Availability Rule record is written with effective_from set to now | Triggering screen shows its Success state ("Hours saved" / "Overrides saved") | FEAT-02.SPEC-001 or FEAT-02.SPEC-002 (triggering screen), FEAT-02.SPEC-004 (conflict check runs against the new version, finding nothing since no confirmed bookings can predate the account's first rule) |
| New version created | An Availability Rule already existed for this Pro Account | A new Availability Rule record is written with effective_from set to now; the prior version is retained unmodified and is no longer the current one | Triggering screen shows its Success state | FEAT-02.SPEC-001 or FEAT-02.SPEC-002, FEAT-02.SPEC-004 (conflict check runs against the new version) |
| Versioning failure | The write cannot be committed (e.g., a connectivity or processing failure after validation passed) | No new version is written; the prior version (if any) remains current and unchanged | Triggering screen shows its Error state with a retry action; the Pro's entered values remain on screen for retry | FEAT-02.SPEC-001 or FEAT-02.SPEC-002 (triggering screen shows the failure) |

## Data Model

**Reads:** Availability Rule -- the current (latest-effective) version's full field set, to carry forward any fields not changed by the triggering screen.
**Creates:** Availability Rule -- a new version with weekly_windows, default_buffer, minimum_booking_notice, booking_horizon, and effective_from; when triggered from FEAT-02.SPEC-002, the corresponding Service.buffer_override field is written as part of the same save (on the Service entity, per the flagged discrepancy noted in feature-overview.md).
**Updates:** None on the Availability Rule entity itself -- every change is a new version, never an in-place update. Marking a version as "current" is a pointer change, not a modification of any existing version's stored field values.
**Deletes:** None -- prior versions are never deleted, per scope-boundaries.md SC-22 and feature-overview.md's Non-Goals.

## Business Rules

- An Availability Rule is never overwritten in place -- every passing save, from either triggering screen, produces a wholly new, additional version (feature-overview.md's Side-Effect Inventory).
- Every prior version is retained indefinitely as long as any booking may reference it (dependency map: "never removed while past bookings reference it"; scope-boundaries.md SC-22).
- Versioning is unconditional on a passing save -- there is no "no-op" outcome where a validated save produces no new version, even when the entered values happen to match the current version exactly, so that effective_from always accurately reflects when the Pro last confirmed their hours.
- Versioning always triggers the conflict check (FEAT-02.SPEC-004) against confirmed bookings; a version is never left unchecked (XBR-11).
- Versioning itself never inspects or changes Booking records -- that is FEAT-02.SPEC-004's role, triggered as a direct consequence of this automation.

## Edge Cases

- **The Pro has never set an Availability Rule and saves for the first time** -- Step 2 finds no current version; the new version is created with no prior fields to carry forward, and FEAT-02.SPEC-004's conflict check against it trivially finds nothing (no confirmed bookings could exist before the account's first rule).
- **A FEAT-02.SPEC-002 save arrives when no Availability Rule exists yet** -- Cannot occur: FEAT-02.SPEC-002 is reached only from FEAT-02.SPEC-001, which requires an Availability Rule (even a freshly created one) to exist first; the per-service override screen has no independent entry point.
- **The triggering screen's entered values are identical to the current version's values** -- A new version is still created with a fresh effective_from, per the unconditional-versioning business rule above.
- **Versioning fails after validation already passed** -- The prior version remains current and unmodified; the triggering screen shows its Error state and the Pro's entered values are preserved for retry, so nothing is left in an inconsistent or partially-versioned state.
- **Concurrent trigger firing (Talia saves from FEAT-02.SPEC-001 on her phone and FEAT-02.SPEC-002 on a second device at nearly the same time)** -- Each save reads whatever version is current at that moment and creates its own new version from it; the save that commits second becomes the latest-effective version and its fields (carrying forward whatever it read as "current" at read time) take precedence, consistent with the dependency map's Contention note ("last-write-wins between the Pro's sessions"). The earlier save's version is retained in history but is no longer current.
- **Trigger fires while a previous run is in flight** -- A second run for the same Pro Account cannot start while the first is in flight: the triggering screens' Save controls are disabled while saving (FEAT-02.SPEC-001, FEAT-02.SPEC-002), so a Pro cannot fire two saves from the same screen session simultaneously. Runs from two different sessions (see the concurrent-trigger-firing entry above) proceed independently, each producing its own version.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Triggered by (inbound), Affects (outbound) | Fires on a passing save; returns success/failure feedback to that screen |
| FEAT-02.SPEC-002 (Per-Service Buffer Override) | Triggered by (inbound), Affects (outbound) | Fires on a passing save; returns success/failure feedback to that screen |
| FEAT-02.SPEC-005 (Availability Setup Validation & Limits) | References (inbound) | This automation only runs after that spec's validation has already passed on the triggering screen |
| FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging) | Triggers (outbound) | Every successfully written new version triggers this conflict check |

## Analytics and Success Signals

- **availability_rule_version_created** (trigger source: FEAT-02.SPEC-001 or FEAT-02.SPEC-002; version sequence number for this account) -- supports success-metrics.md: "Availability Setup Accuracy"
- **availability_rule_versioning_failed** (trigger source, failure stage) -- supports success-metrics.md: "Availability Setup Accuracy" (a versioning failure means the Pro's intended hours never took effect, which is the exact accuracy gap this metric measures)

## Acceptance Criteria

**FEAT-02.SPEC-003-AC-01:** Given Talia has no Availability Rule yet, when she completes and saves the Working Hours screen (FEAT-02.SPEC-001) and validation passes, then a first Availability Rule version is created with effective_from set to the moment of save.

**FEAT-02.SPEC-003-AC-02:** Given Talia has an existing Availability Rule, when she edits her default buffer on FEAT-02.SPEC-001 and validation passes, then a new version is created carrying forward her existing weekly_windows, minimum_booking_notice, and booking_horizon unchanged, with only default_buffer updated.

**FEAT-02.SPEC-003-AC-03:** Given Talia has an existing Availability Rule, when she sets a per-service buffer override on FEAT-02.SPEC-002 and validation passes, then a new version is created that carries forward the existing weekly_windows, default_buffer, minimum_booking_notice, and booking_horizon unchanged, alongside the new Service.buffer_override.

**FEAT-02.SPEC-003-AC-04:** Given a new Availability Rule version has just been written, then FEAT-02.SPEC-004 is triggered against that version before this automation reports success to the triggering screen.

**FEAT-02.SPEC-003-AC-05:** Given Talia re-saves the exact same values already in her current Availability Rule version, when validation passes, then a new version is still created with a fresh effective_from.

**FEAT-02.SPEC-003-AC-06:** Given the write for a new version fails after validation passed, then the prior version remains current and unmodified, and the triggering screen shows its Error state with the Pro's entered values preserved.

**FEAT-02.SPEC-003-AC-07:** Given Talia saves from two sessions (phone and desktop) at nearly the same time, when both saves are validated independently, then the save that commits second becomes the latest-effective version, and the earlier save's version is retained in history but is no longer current.

**FEAT-02.SPEC-003-AC-08:** Given Talia's Working Hours screen has its Save button disabled while a save is in progress, then no second versioning run can start for that same in-flight save.

**FEAT-02.SPEC-003-AC-09:** Given every prior Availability Rule version for Talia's account, when a new version is created, then none of the prior versions are modified or removed.

**FEAT-02.SPEC-003-AC-10:** Given Talia's account has several superseded Availability Rule versions, then none of them are ever automatically purged, regardless of age.

**FEAT-02.SPEC-003-AC-11:** Given a new Availability Rule version is created for Talia's account for the first time (no prior version existed), then the conflict check (FEAT-02.SPEC-004) runs against it and finds no confirmed bookings to flag, since none could predate the account's first rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (FEAT-02.SPEC-001, FEAT-02.SPEC-002) | 2 |
| Outcome Paths | 3 (first version, new version, versioning failure) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Confirmed Booking Conflict Flagging

## Overview

**Name:** Confirmed Booking Conflict Flagging
**ID:** FEAT-02.SPEC-004
**Type:** Automation
**Purpose:** System checks every existing confirmed booking against a newly saved Availability Rule version and flags any that now fall outside working hours, without ever cancelling them.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup

## Scope and Non-Goals

**In Scope:**
- Comparing every upcoming confirmed booking's start time and duration (plus applicable buffer) against a newly saved Availability Rule version's weekly windows
- Producing the set of bookings that no longer fit, for Pro Booking Management (FEAT-30) to surface as attention items
- Never modifying, cancelling, or rescheduling any Booking record as part of this check

**Non-Goals:**
- Cancelling, rescheduling, or otherwise resolving a flagged booking -- excluded per XBR-11: setup changes never silently cancel a confirmed booking; resolution happens only through the Pro's explicit choice in Pro Booking Management (FEAT-30)
- Checking bookings against Time Blocks, Calendar Connection busy time, or Recurring Series -- this spec checks only the newly saved Availability Rule's weekly windows and buffer; the full slot-fitness computation (including those other inputs) belongs to FEAT-03 (Real-Time Slot Availability Engine) and applies only to new bookings, not to re-checking existing ones
- Notifying the client whose booking is flagged -- the Brief's Side-Effect Inventory and XBR-11 route flagged bookings to the Pro's own attention only; any client-facing communication about a resulting change is triggered separately, by whatever action the Pro takes in Pro Booking Management (FEAT-08 messaging)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A new Availability Rule version is written | FEAT-02.SPEC-003 (Availability Rule Versioning) | Fires every time FEAT-02.SPEC-003 successfully commits a new version, whether triggered from FEAT-02.SPEC-001 or FEAT-02.SPEC-002 | The newly written Availability Rule version's full field set (weekly_windows, default_buffer, minimum_booking_notice, booking_horizon) and, when relevant, the changed Service.buffer_override |

## Processing Logic

1. Receive the newly written Availability Rule version from FEAT-02.SPEC-003.
2. Read every Booking for this Pro Account currently in the Confirmed state with a start_time in the future.
3. For each such booking, read its service (to determine the applicable buffer: the service's buffer_override if one is set on the new version, otherwise the new version's default_buffer) and its start_time and duration.
4. For each booking, evaluate whether its start_time through start_time + duration + applicable buffer falls entirely within one of the new version's weekly_windows for that day of week, interpreted in the Pro's account timezone.
5. Classify each evaluated booking as either fitting (no action) or no longer fitting (flagged).
6. For every booking classified as no longer fitting, hand off its reference to Pro Booking Management (FEAT-30) so it appears on that feature's attention list; the Booking record itself is left completely unchanged -- no state transition, no cancellation.
7. If zero bookings are flagged, the run completes silently with no Pro-visible output from this automation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| No conflicts found | Every confirmed future booking still fits the new version | None | None -- the run is silent; the triggering screen's own success confirmation (FEAT-02.SPEC-001 / FEAT-02.SPEC-002) is the only feedback the Pro sees | -- |
| Conflicts found | One or more confirmed future bookings no longer fit the new version | None on the Booking record itself; the flagged reference is handed to Pro Booking Management (FEAT-30) for display | An attention item appears on Pro Booking Management (FEAT-30) per flagged booking; the triggering screen (FEAT-02.SPEC-001 / FEAT-02.SPEC-002) still shows its own ordinary success state, never an error | FEAT-30 (Pro Booking Management) |
| Check failure | The comparison cannot complete (e.g., a processing failure reading bookings) | None | The triggering screen's own save still reports success (versioning already committed); a non-blocking notice appears on the Pro's dashboard: "Some upcoming bookings could not be checked against your new hours -- we'll check again shortly," and the check is retried automatically in the background until it completes | FEAT-02.SPEC-001, FEAT-02.SPEC-002 (unaffected -- their own save already succeeded), Pro Daily Schedule Dashboard (FEAT-12, notice display) |

## Data Model

**Reads:** Booking -- state (Confirmed only), start_time, duration, service reference, for every booking belonging to this Pro Account with a future start_time. Availability Rule -- the newly written version's weekly_windows and default_buffer. Service -- buffer_override, per booking's service, when present.
**Creates:** None.
**Updates:** None -- this automation never writes to the Booking entity, per the dependency map ("it never writes to Booking records itself") and XBR-11.
**Deletes:** None.

## Business Rules

- A confirmed booking is never cancelled, rescheduled, or otherwise modified by this automation -- flagging is purely informational and surfaced through Pro Booking Management (XBR-11).
- Only bookings in the Confirmed state with a future start_time are evaluated; past bookings and bookings already in another state (Completed, No-Show, Cancelled, Rescheduled, Awaiting Outcome) are not re-checked, since a rule change has no bearing on an appointment that has already been kept, missed, or resolved.
- The buffer applied per booking is the service's buffer_override if the new version carries one for that service, otherwise the new version's default_buffer -- the same precedence FEAT-03 applies when computing live slots.
- This check runs against the newly written version only -- it never re-evaluates bookings against any prior, superseded version.
- Resolution of a flagged booking (cancel, reschedule, or keep as an exception) is entirely the Pro's explicit choice, made through Pro Booking Management, never automated here (XBR-11).

## Edge Cases

- **No confirmed future bookings exist for this Pro Account** -- The run completes immediately with zero comparisons and no flags.
- **A booking's service has no buffer_override and the new version's default_buffer changed** -- The booking is evaluated using the new default_buffer; a booking that fit under the old default may newly fail to fit and gets flagged.
- **A booking spans a boundary exactly (start_time + duration + buffer ends exactly at a window's end)** -- Treated as fitting; the fit test is inclusive of the exact boundary, consistent with FEAT-03's own "fits entirely within an open working window" rule.
- **The same booking is flagged by two different rule-version checks in quick succession** (e.g., the Pro edits hours twice within a short span) -- Each check evaluates the booking against its own triggering version independently; the more recent check's flag (or lack of one) is what Pro Booking Management displays, since it reflects the currently-current version.
- **A flagged booking is resolved by the Pro (via FEAT-30) before this automation's next run** -- The next run for a subsequent version change re-evaluates the booking fresh, based on its state at that time; a booking the Pro has since cancelled or rescheduled is simply no longer in the Confirmed-with-future-start_time set this automation reads.
- **Concurrent trigger firing (two Availability Rule versions are written in close succession, e.g., a rapid edit-then-re-edit)** -- Each version triggers its own independent conflict check; if the second version's check completes after the first's, its flag set supersedes the first's for display purposes, since it reflects the latest-effective rule.
- **Trigger fires while a previous run is still in flight** -- A run started for an earlier version is allowed to complete; a run started for a newer version proceeds independently and its results, once ready, are what Pro Booking Management shows, since they reflect the currently-current version. Neither run blocks the other, and neither blocks the triggering screen's own save confirmation, which has already been shown.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-003 (Availability Rule Versioning) | Triggered by (inbound) | Fires on every successfully written new Availability Rule version |
| FEAT-30 (Pro Booking Management) | Affects (outbound) | Flagged bookings are surfaced there as attention items for the Pro's explicit resolution |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A check-failure notice is shown there as a non-blocking attention item |
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | References (inbound) | The triggering screen's own success feedback is unaffected by this automation's outcome |
| FEAT-02.SPEC-002 (Per-Service Buffer Override) | References (inbound) | The triggering screen's own success feedback is unaffected by this automation's outcome |

## Analytics and Success Signals

- **booking_conflict_check_completed** (result: no_conflicts / conflicts_found; conflict_count) -- supports success-metrics.md: "Availability Setup Accuracy" (a rising rate of conflicts found indicates the Pro's edits are surprising her own existing bookings, the exact gap this metric measures)
- **booking_conflict_flagged** (booking reference, reason: outside new working windows) -- supports success-metrics.md: "Availability Setup Accuracy"
- **booking_conflict_check_failed** (retry attempt number) -- N/A -- no success-metrics.md metric measures the reliability of this internal check itself; recorded here as an explicit gap rather than silently dropped, consistent with pipeline-rules.md's output-completeness constraint

## Acceptance Criteria

**FEAT-02.SPEC-004-AC-01:** Given Talia has three confirmed future bookings that all still fit her newly saved hours, when FEAT-02.SPEC-003 triggers this check, then no attention items appear on Pro Booking Management and Talia's save shows only its ordinary success confirmation.

**FEAT-02.SPEC-004-AC-02:** Given Talia narrows her Tuesday working window such that a confirmed Tuesday booking no longer fits, when this check runs against the new version, then that booking is flagged and appears as an attention item on Pro Booking Management, while the booking itself remains Confirmed and unchanged.

**FEAT-02.SPEC-004-AC-03:** Given a flagged booking, when Talia views it on Pro Booking Management, then she sees the attention item and can choose to cancel, reschedule, or keep it as an exception -- this automation itself takes no such action.

**FEAT-02.SPEC-004-AC-04:** Given Talia has a confirmed booking already in the Completed state, when a new Availability Rule version is saved, then that booking is not evaluated or flagged by this check.

**FEAT-02.SPEC-004-AC-05:** Given Talia sets a per-service buffer override that increases the buffer a service's confirmed booking needs, when this check runs, then the booking is evaluated with the new override applied and is flagged if it no longer fits.

**FEAT-02.SPEC-004-AC-06:** Given a confirmed booking whose end time plus buffer lands exactly at the edge of a working window, when this check runs, then the booking is treated as fitting.

**FEAT-02.SPEC-004-AC-07:** Given this check cannot complete due to a processing failure, then Talia's triggering save still shows its own success confirmation, and a non-blocking notice appears on her dashboard that some bookings could not yet be checked, with an automatic retry.

**FEAT-02.SPEC-004-AC-08:** Given Talia has zero confirmed future bookings, when a new Availability Rule version is saved, then this check completes immediately with no flags.

**FEAT-02.SPEC-004-AC-09:** Given Talia edits her hours twice in quick succession, when both versions trigger their own conflict checks, then the flag set shown on Pro Booking Management reflects the check against the most recently saved version.

**FEAT-02.SPEC-004-AC-10:** Given a booking was flagged and Talia resolves it through Pro Booking Management before her next hours edit, when she saves another new version, then that booking is re-evaluated fresh based on its current state, not its prior flag.

**FEAT-02.SPEC-004-AC-11:** Given this automation runs for a Pro Account whose Availability Rule was just created for the first time, when the check runs, then it finds no confirmed bookings to flag, since none could exist before the account's first rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (new Availability Rule version written) | 1 |
| Outcome Paths | 3 (no conflicts, conflicts found, check failure) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Availability Setup Validation & Limits

## Overview

**Name:** Availability Setup Validation & Limits
**ID:** FEAT-02.SPEC-005
**Type:** Logic/Rule
**Purpose:** Defines every validation and boundary rule governing the Availability Rule and its per-service buffer override, plus authorization rules for every action on both, shared by FEAT-02.SPEC-001 and FEAT-02.SPEC-002 so neither screen duplicates the rules.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup
**Governed Entity:** Availability Rule (weekly_windows, default_buffer, minimum_booking_notice, booking_horizon, effective_from), plus the Service entity's buffer_override field, which this feature's rules also govern even though the field is stored on Service (flagged discrepancy, per feature-overview.md's Entity-Lifecycle Coverage Matrix).

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for every Availability Rule field and for Service.buffer_override
- Cross-field rules (window ordering, no-overlap, timezone interpretation)
- Authorization rules for every action on the Availability Rule and on Service.buffer_override, for every role in the Access Matrix
- Default values and derived fields (effective_from)
- Error messages for every validation failure

**Non-Goals:**
- Validating any other Service field (name, price, duration, deposit_rule) -- owned by FEAT-01's own validation, not this spec, which addresses buffer_override only
- Computing whether a specific slot is actually bookable (fitting duration, buffer, notice, and horizon together against live bookings, blocks, and calendar busy time) -- owned by FEAT-03 (Real-Time Slot Availability Engine); this spec defines the boundary values Talia may set, not the live computation that consumes them
- Checking existing confirmed bookings against a newly saved rule -- owned by FEAT-02.SPEC-004, which is triggered only after this spec's validation has already passed
- Setting or validating the Pro's account timezone itself -- excluded per XBR-25: FEAT-27 is the sole owner of timezone; this spec only reads it to interpret entered times

## Governed Entity

**Entity:** Availability Rule (plus Service.buffer_override, set through this feature's screens)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| weekly_windows | text (structured) | Per day of week, one or more non-overlapping start/end time pairs; interpreted in the Pro's account timezone |
| default_buffer | number | Minutes applied between consecutive bookings when no per-service override applies |
| minimum_booking_notice | number | How close to an appointment a client may still book, in days |
| booking_horizon | number | How far ahead a client may book, in weeks or months |
| effective_from | date | The date/time from which this version applies -- system-derived, not user-entered |
| Service.buffer_override (referenced) | number | Optional per-service override of default_buffer, 0 to 120 minutes; stored on the Service entity, set only through this feature's screens |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-001 | Working Hours, Buffer, Notice & Horizon Setup | On field blur and form submit for weekly_windows, default_buffer, minimum_booking_notice, booking_horizon; authorization on screen entry and on save |
| FEAT-02.SPEC-002 | Per-Service Buffer Override | On field blur and form submit for Service.buffer_override; authorization on screen entry and on save |
| FEAT-02.SPEC-003 | Availability Rule Versioning | Runs only after this spec's validation has passed on the triggering screen -- re-derives effective_from at commit time |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| weekly_windows (per window) | Start time must be before end time | Always, for every entered window | On blur | "End time must be after start time" | Yes |
| weekly_windows (per day) | No two windows on the same day may overlap | Always, when a day has 2 or more windows | On blur (of the second window's fields) and on submit | "This window overlaps with another window on the same day" | Yes |
| weekly_windows | At least one window must exist on at least one day before the account's first save can complete (an Availability Rule with zero windows can never make the booking link go live, per XBR-26) | Only on the very first save for this account (the Empty state) | On submit | "Add at least one working window before saving" | Yes |
| default_buffer | Whole number of minutes, 0 to 120 inclusive | Always | On blur | "Buffer must be between 0 and 120 minutes" | Yes |
| minimum_booking_notice | Whole number of days, 0 to 7 inclusive | Always | On blur | "Minimum notice must be between 0 and 7 days" | Yes |
| booking_horizon | Whole number, 1 week to 12 months inclusive (stored and compared in weeks: 1 to 52) | Always | On blur | "Booking horizon must be between 1 week and 12 months" | Yes |
| effective_from | No validation beyond data type -- system-derived, never user-entered | Always | -- | -- | -- |
| Service.buffer_override | Whole number of minutes, 0 to 120 inclusive, same bounds as default_buffer | Only when the Pro sets an override (an empty override is valid and falls back to default_buffer) | On blur | "Override must be between 0 and 120 minutes" | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Window ordering and no-overlap is evaluated per day, across all windows for that day | weekly_windows (all windows within one day) | Every window on a given day must individually pass the start-before-end rule, and no two windows on that day may share any overlapping time span, including a zero-length gap treated as non-overlapping (e.g., 9:00-12:00 and 12:00-15:00 are valid adjacent windows, not an overlap) | "This window overlaps with another window on the same day" |
| All entered and displayed times are interpreted in the Pro's account timezone | weekly_windows, Pro Account.timezone (read-only reference) | Every start/end time the Pro enters is stored and interpreted against the Pro Account's current timezone (FEAT-27); the screen displays the active timezone label alongside the fields rather than asking the Pro to specify it here | N/A -- this is an interpretation rule, not a rejectable input; there is no invalid-timezone state on this screen since timezone is never entered here |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|---------------------------------------------|
| Create the initial Availability Rule (first save) | The Pro (Talia) | Always | -- |
| View the current Availability Rule (weekly hours, buffer, notice, horizon) | The Pro (Talia) | Always (own account only) | -- |
| Update the Availability Rule (create a new version) | The Pro (Talia) | Always (own account only) | -- |
| Set or clear a Service.buffer_override | The Pro (Talia) | Always, for the Pro's own active services only | -- |
| View the current Availability Rule or Service.buffer_override | The Client (Riley) | Never | No client-facing screen or field exposes any part of the Availability Rule or a service's buffer_override at any time; the client sees only the resulting open times computed by FEAT-03, never the rule itself |
| Update the Availability Rule or Service.buffer_override | The Client (Riley) | Never | No client-facing control exists to attempt this action |
| View the current Availability Rule or Service.buffer_override | Platform Operator (Support) | Always, read-only, for troubleshooting only | -- |
| Update the Availability Rule or Service.buffer_override | Platform Operator (Support) | Never | All edit and save controls are hidden on both FEAT-02.SPEC-001 and FEAT-02.SPEC-002 when viewed by Support; a direct attempt through a stale or replayed request is refused with "Support access is read-only and cannot make changes to this account." (XBR-24) |
| Any action on the Availability Rule or Service.buffer_override | Unauthenticated visitor | Never | Redirected to the Pro sign-in screen (FEAT-29); a failed sign-in never reveals whether an account exists (XBR-29) |
| View or update the Availability Rule or Service.buffer_override with an expired session | The Pro (Talia) | Never, until re-authenticated | Dialog: "Your session has expired. Sign in to continue."; any unsaved edits are preserved and restored after successful re-authentication |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| effective_from | Set to the exact moment the triggering save commits | On every create and every new version (FEAT-02.SPEC-003) | No -- always system-derived, never user-entered |
| minimum_booking_notice (initial value, before the Pro has ever set one) | Defaults to 0 days, per product-features.md's stated default of "a few hours" -- this field is validated and stored only in whole days (0 to 7, per the Field Validation Rules table above), and a span of a few hours is less than one whole day, so it resolves to the field's minimum representable value, 0 days, at this field's day-level granularity | On the account's first Availability Rule creation only, shown as the pre-filled value the Pro can change before her first save | Yes -- the Pro may change it before or at any point after the first save |
| booking_horizon (initial value, before the Pro has ever set one) | Defaults to 8 weeks, per product-features.md's stated default | On the account's first Availability Rule creation only, shown as the pre-filled value the Pro can change before her first save | Yes -- the Pro may change it before or at any point after the first save |
| default_buffer (initial value, before the Pro has ever set one) | Defaults to 0 minutes -- no gap is assumed until the Pro sets one | On the account's first Availability Rule creation only | Yes |
| Service.buffer_override (initial value, per service) | No override -- falls back to the current Availability Rule's default_buffer until the Pro explicitly sets one | Always, for a service with no override on record | Yes -- setting an override at any time takes precedence over the default until cleared |

## Business Rules

- Minimum booking notice and booking horizon limit every client-facing booking path computed by FEAT-03; the Pro alone may book inside notice or beyond horizon through Pro Booking Management (XBR-03) -- this exception is enforced by FEAT-30, not by this spec, which governs only what Talia may enter here.
- All hours are interpreted in the Pro's account timezone, which this feature reads but never sets; a timezone change is owned entirely by FEAT-27 (XBR-25).
- A Service.buffer_override uses the exact same numeric bounds as default_buffer (0 to 120 minutes) so the two fields behave identically wherever either is displayed (feature-overview.md's Shared Context: "same bounds validation... described identically across both screens").
- These validation and authorization rules apply identically whether the save originates from FEAT-02.SPEC-001 or FEAT-02.SPEC-002 -- the product definition establishes no screen-specific exception to any rule in this spec.
- A validated save always proceeds to FEAT-02.SPEC-003 (versioning) and, through it, to FEAT-02.SPEC-004 (conflict flagging); this spec's rules gate entry into that pipeline but do not themselves check existing bookings.

## Edge Cases

- **A window is entered with start time exactly equal to end time** -- Rejected: end time must be strictly after start time, not merely equal.
- **Two windows on the same day share exactly one boundary instant** (e.g., 9:00-12:00 and 12:00-15:00) -- Accepted as non-overlapping adjacent windows, not rejected.
- **default_buffer entered as exactly 0** -- Accepted; a zero buffer is valid and means back-to-back bookings are allowed.
- **default_buffer entered as exactly 120** -- Accepted (the upper bound); 121 is rejected.
- **minimum_booking_notice entered as exactly 0** -- Accepted; a client may book right up to the appointment time.
- **minimum_booking_notice entered as exactly 7 (days)** -- Accepted (the upper bound); 8 is rejected.
- **booking_horizon entered as exactly 1 week or exactly 12 months (52 weeks)** -- Both boundary values are accepted; anything outside that range is rejected.
- **Service.buffer_override entered as exactly 0 or exactly 120** -- Both boundary values are accepted, identically to default_buffer's own bounds.
- **The Pro clears a Service.buffer_override entirely (empty field) rather than entering 0** -- Treated as "no override" (falls back to default_buffer), distinct from an explicit override of 0 minutes, which is itself a valid, stored value.
- **The Pro's account timezone changes (via FEAT-27) between when a window's times were entered and when the save commits** -- The save commits using the timezone current at commit time; the screen (FEAT-02.SPEC-001) surfaces a one-time notice that the displayed label has updated, per that spec's own edge cases.
- **A Support view attempts to invoke the save action directly (bypassing the hidden controls)** -- Refused with "Support access is read-only and cannot make changes to this account." regardless of the values submitted.
- **The Pro's session expires mid-edit with valid but unsaved values on screen** -- No validation or save occurs until re-authentication succeeds; the unsaved values are preserved and re-validated against the current rules once the Pro is signed back in.

## Acceptance Criteria

**FEAT-02.SPEC-005-AC-01:** Given Talia enters a window with start time 2:00 PM and end time 1:00 PM on the same day, when the field loses focus, then the error "End time must be after start time" appears and the window cannot be saved.

**FEAT-02.SPEC-005-AC-02:** Given Talia enters a window with start time 9:00 AM and end time 12:00 PM, when the field loses focus, then no error appears.

**FEAT-02.SPEC-005-AC-03:** Given Talia has a 9:00 AM-1:00 PM window on Monday and adds a second window of 12:00 PM-3:00 PM on Monday, when she attempts to save, then the error "This window overlaps with another window on the same day" appears and the save does not proceed.

**FEAT-02.SPEC-005-AC-04:** Given Talia has a 9:00 AM-12:00 PM window on Monday and adds a second window of exactly 12:00 PM-3:00 PM, when she attempts to save, then no overlap error appears, since the windows are adjacent, not overlapping.

**FEAT-02.SPEC-005-AC-05:** Given Talia has never set any working window, when she attempts to save with zero windows across every day, then the error "Add at least one working window before saving" appears and the save does not proceed.

**FEAT-02.SPEC-005-AC-06:** Given Talia enters 121 as her default buffer, when the field loses focus, then the error "Buffer must be between 0 and 120 minutes" appears.

**FEAT-02.SPEC-005-AC-07:** Given Talia enters 120 as her default buffer, when the field loses focus, then no error appears and the value is accepted.

**FEAT-02.SPEC-005-AC-08:** Given Talia enters 8 as her minimum booking notice in days, when the field loses focus, then the error "Minimum notice must be between 0 and 7 days" appears.

**FEAT-02.SPEC-005-AC-09:** Given Talia enters 0 as her minimum booking notice, when the field loses focus, then no error appears and the value is accepted.

**FEAT-02.SPEC-005-AC-10:** Given Talia enters a booking horizon of 13 months, when the field loses focus, then the error "Booking horizon must be between 1 week and 12 months" appears.

**FEAT-02.SPEC-005-AC-11:** Given Talia enters a booking horizon of exactly 1 week, then no error appears and the value is accepted.

**FEAT-02.SPEC-005-AC-12:** Given Talia sets a per-service buffer override of 130 minutes on FEAT-02.SPEC-002, when the field loses focus, then the error "Override must be between 0 and 120 minutes" appears.

**FEAT-02.SPEC-005-AC-13:** Given Talia sets a per-service buffer override of exactly 0 minutes, then it is accepted and stored as an explicit override distinct from having no override at all.

**FEAT-02.SPEC-005-AC-14:** Given Talia (the Pro) is the sole role interacting with this spec's screens, when she saves valid values, then the save proceeds without any authorization check blocking her.

**FEAT-02.SPEC-005-AC-15:** Given Riley (the Client) has no screen or control anywhere in the product that exposes any Availability Rule field or a service's buffer_override, then Riley can never view or update either.

**FEAT-02.SPEC-005-AC-16:** Given Platform Operator (Support) opens Talia's account, when Support views the Availability Rule or Service.buffer_override, then all values are shown read-only with no edit or save controls present.

**FEAT-02.SPEC-005-AC-17:** Given a stale or replayed save request reaches the system while flagged as a Support-originated action, then it is refused with "Support access is read-only and cannot make changes to this account." regardless of the values submitted.

**FEAT-02.SPEC-005-AC-18:** Given a visitor who is not signed in as a Pro attempts to view or update the Availability Rule, then they are redirected to the Pro sign-in screen without any indication of whether an account exists.

**FEAT-02.SPEC-005-AC-19:** Given Talia's session expires with valid but unsaved values entered, when she signs back in, then her unsaved values are restored and re-validated against these same rules before saving.

**FEAT-02.SPEC-005-AC-20:** Given a brand-new Pro Account with no Availability Rule yet, when the Working Hours screen loads for the first time, then minimum_booking_notice is pre-filled at 0 days (product-features.md's "a few hours" resolved to this field's minimum whole-day value) and booking_horizon is pre-filled at 8 weeks, and default_buffer is pre-filled at 0.

**FEAT-02.SPEC-005-AC-21:** Given Talia saves her first Availability Rule, when the save commits, then effective_from is set to the exact moment of commit and cannot be edited by Talia.

**FEAT-02.SPEC-005-AC-22:** Given a service with no buffer_override set, when the current Availability Rule's default_buffer changes, then that service's effective buffer follows the new default automatically, since no override is on record for it.

**FEAT-02.SPEC-005-AC-23:** Given all entered window times, when they are displayed anywhere on FEAT-02.SPEC-001, then they are labeled and interpreted using the Pro Account's current timezone as set by FEAT-27, never a hard-coded or separately entered timezone.

**FEAT-02.SPEC-005-AC-24:** Given Talia clears a previously set Service.buffer_override, when she saves, then the field is stored as having no override (not as 0), and the service's effective buffer reverts to following the account default.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 12 | 12 |

