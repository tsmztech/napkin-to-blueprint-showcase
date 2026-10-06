---
document_type: feature-overview
feature_number: FEAT-02
feature_name: Availability & Working Hours Setup
feature_slug: availability-working-hours-setup
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 5
screen_count: 2
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

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
